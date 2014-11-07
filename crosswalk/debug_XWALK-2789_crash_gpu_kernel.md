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
