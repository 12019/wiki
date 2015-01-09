### Launch
```
#0  xwalk::application::ApplicationSystemTizen::LaunchFromCommandLine (this=, cmd_line=..., url=...) at ../../xwalk/application/browser/application_system_tizen.cc:100
#1  xwalk::XWalkBrowserMainParts::PreMainMessageLoopRun (this=this@entry=) at ../../xwalk/runtime/browser/xwalk_browser_main_parts.cc:221
#2  xwalk::XWalkBrowserMainPartsTizen::PreMainMessageLoopRun (this=) at ../../xwalk/runtime/browser/xwalk_browser_main_parts_tizen.cc:61
#3  content::BrowserMainLoop::PreMainMessageLoopRun (this=) at ../../content/browser/browser_main_loop.cc:743
#4  Run (object=, this=<synthetic pointer>) at ../../base/bind_internal.h:134
#5  MakeItSo (a1=, runnable=...) at ../../base/bind_internal.h:863
#6  base::internal::Invoker<1, base::internal::BindState<base::internal::RunnableAdapter<int (content::BrowserMainLoop::*)()>, int (content::BrowserMainLoop*), void (base::internal::Unretain
edWrapper<content::BrowserMainLoop>)>, int (content::BrowserMainLoop*)>::Run(base::internal::BindStateBase*) (base=) at ../../base/bind_internal.h:1169
#7  Run (this=) at ../../base/callback.h:401
#8  content::StartupTaskRunner::RunAllTasksNow (this=) at ../../content/browser/startup_task_runner.cc:45
#9  content::BrowserMainLoop::CreateStartupTasks (this=) at ../../content/browser/browser_main_loop.cc:641
#10 content::BrowserMainRunnerImpl::Initialize (this=, parameters=...) at ../../content/browser/browser_main_runner.cc:230
#11 content::BrowserMain (parameters=...) at ../../content/browser/browser_main.cc:22
#12 content::RunNamedProcessTypeMain (process_type=..., main_function_params=..., delegate=) at ../../content/app/content_main_runner.cc:423
#13 content::ContentMainRunnerImpl::Run (this=) at ../../content/app/content_main_runner.cc:798
#14 content::ContentMain (params=...) at ../../content/app/content_main.cc:19
#15 main (argc=2, argv=) at ../../xwalk/runtime/app/xwalk_main.cc:40
```


### CreateWindowTreeHost

```
#0  views::DesktopWindowTreeHostWayland::DesktopWindowTreeHostWayland (this=, native_widget_delegate=, desktop_native_widget_aura=)
#1  views::DesktopFactoryWayland::CreateWindowTreeHost (this=<optimized out>, native_widget_delegate=, desktop_native_widget_aura=)
#2  views::DesktopNativeWidgetAura::InitNativeWidget (this=, params=...) at ../../ui/views/widget/desktop_aura/desktop_native_widget_aura.cc:418
#3  views::Widget::Init (this=, in_params=...) at ../../ui/views/widget/widget.cc:360
#4  xwalk::NativeAppWindowViews::Initialize (this=) at ../../xwalk/runtime/browser/ui/native_app_window_views.cc:66
#5  xwalk::NativeAppWindowTizen::Initialize (this=) at ../../xwalk/runtime/browser/ui/native_app_window_tizen.cc:100
#6  xwalk::NativeAppWindow::Create (create_params=...) at ../../xwalk/runtime/browser/ui/native_app_window_views.cc:303
#7  xwalk::Runtime::AttachWindow (this=, params=...) at ../../xwalk/runtime/browser/runtime.cc:130
#8  xwalk::application::Application::Launch (this=<optimized out>, launch_params=...)
#9  xwalk::application::ApplicationTizen::Launch (this=, launch_params=...) at ../../xwalk/application/browser/application_tizen.cc:135
#10 xwalk::application::ApplicationService::Launch (this=this@entry=, application_data=..., launch_params=...) at ../../xwalk/application/browser/application_service.cc:70
#11 xwalk::application::ApplicationService::LaunchHostedURL (this=, url=..., params=...) at ../../xwalk/application/browser/application_service.cc:177
#12 xwalk::application::ApplicationSystem::LaunchFromCommandLine (this=, cmd_line=..., url=..., run_default_message_loop=true) at ../../xwalk/application/browser/application_system.cc:91
#13 xwalk::XWalkBrowserMainParts::PreMainMessageLoopRun (this=)
#14 content::BrowserMainLoop::PreMainMessageLoopRun (this=)
#15 Run (this=) at ../../base/callback.h:401
#16 content::StartupTaskRunner::RunAllTasksNow (this=)
#17 content::BrowserMainLoop::CreateStartupTasks (this=)
#18 content::BrowserMainRunnerImpl::Initialize (this=, parameters=...) at ../../content/browser/browser_main_runner.cc:193
#19 content::BrowserMain (parameters=...) at ../../content/browser/browser_main.cc:22
#20 content::ContentMainRunnerImpl::Run (this=)
#21 content::ContentMain (params=...) at ../../content/app/content_main.cc:19
#22 main (argc=2, argv=) at ../../xwalk/runtime/app/xwalk_main.cc:40
```

### EstablishGpuChannel
```
#0  content::BrowserGpuChannelHostFactory::EstablishGpuChannel(content::CauseForGpuLaunch, base::Callback<void ()> const&) (this=, cause_for_gpu_launch=cause_for_gpu_launch@entry=content::CAUSE_FOR_GPU_LAUNCH_BROWSER_STARTUP, callback=...) at ../../content/browser/gpu/browser_gpu_channel_host_factory.cc:344
#1  content::BrowserGpuChannelHostFactory::Initialize (establish_gpu_channel=<optimized out>) at ../../content/browser/gpu/browser_gpu_channel_host_factory.cc:233
#2  content::BrowserMainLoop::BrowserThreadsStarted (this=)
#3  Run (this=)
#4  content::StartupTaskRunner::RunAllTasksNow (this=)
#5  content::BrowserMainLoop::CreateStartupTasks (this=) at ../../content/browser/browser_main_loop.cc:633
#6  content::BrowserMainRunnerImpl::Initialize (this=, parameters=...)
#7  content::BrowserMain (parameters=...) at ../../content/browser/browser_main.cc:22
#8  content::ContentMainRunnerImpl::Run (this=) at ../../content/app/content_main_runner.cc:778
#9  content::ContentMain (params=...) at ../../content/app/content_main.cc:19
#10 main (argc=2, argv=)
```

### Display::~Diplay
Calling from the browser process
```
(gdb) bt
#0  gfx::Display::~Display (this=, __in_chrg=<optimized out>) at ../../ui/gfx/display.cc:85
#1  views::DesktopScreenWayland::SetGeometry (this=, geometry=...) at ../../ozone/ui/desktop_aura/desktop_screen_wayland.cc:38
#2  views::DesktopScreenWayland::OnOutputSizeChanged (this=<optimized out>, width=<optimized out>, height=<optimized out>) at ../../ozone/ui/desktop_aura/desktop_screen_wayland.cc:149
#3  ozonewayland::WaylandDisplay::LookAheadOutputGeometry (this=) at ../../ozone/wayland/display.cc:306
#4  ui::(anonymous namespace)::OzonePlatformWayland::CreatePlatformWindow (this=<optimized out>, delegate=, bounds=...) at ../../ozone/platform/ozone_platform_wayland.cc:61
#5  views::DesktopFactoryWayland::CreateDesktopScreen (this=) at ../../ozone/ui/desktop_aura/desktop_factory_wayland.cc:36
#6  xwalk::NativeAppWindow::Initialize () at ../../xwalk/runtime/browser/ui/native_app_window_views.cc:311
#7  xwalk::XWalkBrowserMainParts::PreMainMessageLoopRun (this=) at ../../xwalk/runtime/browser/xwalk_browser_main_parts.cc:210
#8  content::BrowserMainLoop::PreMainMessageLoopRun (this=) at ../../content/browser/browser_main_loop.cc:736
#9  Run (this=) at ../../base/callback.h:401
#10 content::StartupTaskRunner::RunAllTasksNow (this=) at ../../content/browser/startup_task_runner.cc:45
#11 content::BrowserMainLoop::CreateStartupTasks (this=) at ../../content/browser/browser_main_loop.cc:636
#12 content::BrowserMainRunnerImpl::Initialize (this=, parameters=...) at ../../content/browser/browser_main_runner.cc:193
#13 content::BrowserMain (parameters=...) at ../../content/browser/browser_main.cc:22
#14 content::ContentMainRunnerImpl::Run (this=) at ../../content/app/content_main_runner.cc:773
#15 content::ContentMain (params=...) at ../../content/app/content_main.cc:19
#16 main (argc=3, argv=) at ../../xwalk/runtime/app/xwalk_main.cc:40
(gdb) 
```
