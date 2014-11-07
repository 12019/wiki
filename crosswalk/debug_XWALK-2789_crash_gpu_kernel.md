This is a note for debugging XWALK-2789.
https://crosswalk-project.org/jira/browse/XWALK-2789

It is not easy to attach the GPU porcess in gdb because the brower process create serveral processes so it is good to set "set follow-fork-mode child" in gdb when breaking at void BrowserChildProcessHostImpl::Launch

```
alice@common_box:~$ ps ajxf | grep xwalk
 2317  2364  2363  2317 pts/4     2363 S+    5001   0:00          \_ grep xwalk
 2219  2281  2281  2130 pts/0     2281 S+    5001   0:04              \_ cgdb --arg /home/abuild/rpmbuild/BUILD/crosswalk/src/out/Release/xwalk --disable-gpu-watchdog http://google.com
 2281  2282  2282  2282 pts/2     2282 Ss+   5001   5:21                  \_ gdb --nw --annotate=2 -x /home/alice/.tgdb/a2_gdb_init --arg /home/abuild/rpmbuild/BUILD/crosswalk/src/out/Release/xwalk --disable-gpu-watchdog http://google.com
 2282  2284  2284  2284 pts/3     2284 Ssl+  5001   0:00                      \_ /home/abuild/rpmbuild/BUILD/crosswalk/src/out/Release/xwalk --disable-gpu-watchdog http://google.com
 2284  2290  2284  2284 pts/3     2284 S+    5001   0:00                          \_ /home/abuild/rpmbuild/BUILD/crosswalk/src/out/Release/xwalk --type=zygote
 2290  2355  2284  2284 pts/3     2284 Sl+   5001   0:00                          |   \_ /home/abuild/rpmbuild/BUILD/crosswalk/src/out/Release/xwalk --type=renderer --enable-threaded-compositing --enable-viewport --js-flags=--simd_object --use-gl=egl --enable-deferred-image-decoding --lang=en-US --enable-pinch --enable-delegated-renderer --enable-impl-side-painting --enable-gpu-rasterization --channel=2284.1.2050979475
 2284  2350  2284  2284 pts/3     2284 t+    5001   0:00                          \_ /home/abuild/rpmbuild/BUILD/crosswalk/src/out/Release/xwalk --type=gpu-process --channel=2284.0.352238446 --disable-gpu-watchdog --gpu-no-context-lost --use-gl=egl --supports-dual-gpus=false --gpu-driver-bug-workarounds=1,11,15 --gpu-vendor-id=0x8086 --gpu-device-id=0x0f31 --gpu-driver-vendor --gpu-driver-version
 2284  2357  2284  2284 pts/3     2284 Sl+   5001   0:00                          \_ /home/abuild/rpmbuild/BUILD/crosswalk/src/out/Release/xwalk --type=xwalk-extension-process --channel=2284.2.1208256788
alice@common_box:~$ 
```

```c++
(gdb) bt
#0  ozonewayland::WaylandShell::WaylandShell (this=) at ../../ozone/wayland/shell/shell.cc:17
#1  ozonewayland::WaylandDisplay::InitializeDisplay (this=this@entry=) at ../../ozone/wayland/display.cc:332
#2  ozonewayland::WaylandDisplay::InitializeHardware (this=) at ../../ozone/wayland/display.cc:85
#3  ui::(anonymous namespace)::OzonePlatformWayland::InitializeGPU (this=) at ../../ozone/platform/ozone_platform_wayland.cc:91
#4  gfx::InitializeStaticGLBindings (implementation=gfx::kGLImplementationEGLGLES2) at ../../ui/gl/gl_implementation_ozone.cc:41
#5  gfx::GLSurface::InitializeOneOffImplementation (impl=<optimized out>, fallback_to_osmesa=<optimized out>, gpu_service_logging=<optimized out>, disable_gl_drawing=<optimized out>) at ../../ui/gl/gl_surface.cc:
78
#6  gfx::GLSurface::InitializeOneOff () at ../../ui/gl/gl_surface.cc:69
#7  content::GpuMain (parameters=...) at ../../content/gpu/gpu_main.cc:256
#8  content::ContentMainRunnerImpl::Run (this=) at ../../content/app/content_main_runner.cc:773
#9  content::ContentMain (params=...) at ../../content/app/content_main.cc:19
#10 main (argc=12, argv=) at ../../xwalk/runtime/app/xwalk_main.cc:40

```

```
(gdb) bt
#0  content::LinuxSandbox::InitializeSandboxImpl (this=) at ../../content/common/sandbox_linux/sandbox_linux.cc:272
#1  content::(anonymous namespace)::StartSandboxLinux (gpu_info=..., watchdog_thread=watchdog_thread@entry=warning: (Internal error: pc 0x0 in read in psymtab, but not in symtab.)

, should_initialize_gl_context=<optimized out>) at ../../content/gpu/gpu_main.cc:493
#2  content::GpuMain (parameters=...) at ../../content/gpu/gpu_main.cc:320
#3  content::ContentMainRunnerImpl::Run (this=) at ../../content/app/content_main_runner.cc:773
#4  content::ContentMain (params=...) at ../../content/app/content_main.cc:19
#5  main (argc=12, argv=) at ../../xwalk/runtime/app/xwalk_main.cc:40
```

It seems that the GPU Process crashes here
```C++
bool LinuxSandbox::InitializeSandboxImpl() { 
  base::CommandLine* command_line = base::CommandLine::ForCurrentProcess(); 
  const std::string process_type = 
      command_line->GetSwitchValueASCII(switches::kProcessType); 
 
  // We need to make absolutely sure that our sandbox is "sealed" before 
  // returning. 
  // Unretained() since the current object is a Singleton. 
  base::ScopedClosureRunner sandbox_sealer( 
      base::Bind(&LinuxSandbox::SealSandbox, base::Unretained(this))); 
  // Make sure that this function enables sandboxes as promised by GetStatus(). 
  // Unretained() since the current object is a Singleton. 
  base::ScopedClosureRunner sandbox_promise_keeper( 
      base::Bind(&LinuxSandbox::CheckForBrokenPromises, 
                 base::Unretained(this), 
                 process_type)); 
 
  // No matter what, it's always an error to call InitializeSandbox() after 
  // threads have been created. 
  if (!IsSingleThreaded()) { 
    std::string error_message = "InitializeSandbox() called with multiple " 
                                "threads in process " + process_type; 
    // TSAN starts a helper thread, so we don't start the sandbox and don't 
    // even report an error about it. 
    if (IsRunningTSAN()) 
      return false; 
 
    // The GPU process is allowed to call InitializeSandbox() with threads. 
    bool sandbox_failure_fatal = process_type != switches::kGpuProcess; 
    // This can be disabled with the '--gpu-sandbox-failures-fatal' flag. 
    // Setting the flag with no value or any value different than 'yes' or 'no' 
    // is equal to setting '--gpu-sandbox-failures-fatal=yes'. 
    if (process_type == switches::kGpuProcess && 
        command_line->HasSwitch(switches::kGpuSandboxFailuresFatal)) { 
      const std::string switch_value = 
          command_line->GetSwitchValueASCII(switches::kGpuSandboxFailuresFatal); 
      sandbox_failure_fatal = switch_value != "no"; 
    }    
 
    if (sandbox_failure_fatal) 
      LOG(FATAL) << error_message;
 
    LOG(ERROR) << error_message;
    return false;
```

### Sandbox check in the GPU process
```
(gdb) bt
#0  content::GpuProcessPolicy::PreSandboxHook (this=) at ../../content/common/sandbox_linux/bpf_gpu_policy_linux.cc:275
#1  StartBPFSandbox (command_line=..., process_type=...) at ../../content/common/sandbox_linux/sandbox_seccomp_bpf_linux.cc:194
#2  content::SandboxSeccompBPF::StartSandbox (process_type=...) at ../../content/common/sandbox_linux/sandbox_seccomp_bpf_linux.cc:268
#3  content::LinuxSandbox::StartSeccompBPF (this=this@entry=, process_type=...) at ../../content/common/sandbox_linux/sandbox_linux.cc:255
#4  content::LinuxSandbox::InitializeSandboxImpl (this=) at ../../content/common/sandbox_linux/sandbox_linux.cc:321
#5  content::(anonymous namespace)::StartSandboxLinux (gpu_info=..., watchdog_thread=watchdog_thread@entry=warning: (Internal error: pc 0x0 in read in psymtab, but not in symtab.)

, should_initialize_gl_context=<optimized out>) at ../../content/gpu/gpu_main.cc:493
#6  content::GpuMain (parameters=...) at ../../content/gpu/gpu_main.cc:320
#7  content::ContentMainRunnerImpl::Run (this=) at ../../content/app/content_main_runner.cc:773
#8  content::ContentMain (params=...) at ../../content/app/content_main.cc:19
#9  main (argc=12, argv=) at ../../xwalk/runtime/app/xwalk_main.cc:40
```

### Here is the crash point
```c++
// If a BPF policy is engaged for |process_type|, run a few sanity checks. 
void RunSandboxSanityChecks(const std::string& process_type) { 
  if (process_type == switches::kRendererProcess || 
      process_type == switches::kGpuProcess || 
      process_type == switches::kPpapiPluginProcess) { 
    int syscall_ret; 
    errno = 0; 
 
    // Without the sandbox, this would EBADF. 
    syscall_ret = fchmod(-1, 07777); 
    CHECK_EQ(-1, syscall_ret);    
    CHECK_EQ(EPERM, errno);   <== Here??
 
    // Run most of the sanity checks only in DEBUG mode to avoid a perf. 
    // impact. 
#if !defined(NDEBUG) 
    // open() must be restricted. 
    syscall_ret = open("/etc/passwd", O_RDONLY); 
    CHECK_EQ(-1, syscall_ret); 
    CHECK_EQ(SandboxBPFBasePolicy::GetFSDeniedErrno(), errno); 
 
    // We should never allow the creation of netlink sockets. 
    syscall_ret = socket(AF_NETLINK, SOCK_DGRAM, 0);  
    CHECK_EQ(-1, syscall_ret); 
    CHECK_EQ(EPERM, errno); 
#endif  // !defined(NDEBUG) 
  } 
} 
```
