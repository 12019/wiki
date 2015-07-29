# Crosswalk Wiki

## Build

### Android Build
```
$ cd git/chromium/src
$ export XWALK_OS_ANDROID=1
$ sudo ./build/install-build-deps-android.sh
$ echo "{ 'GYP_DEFINES': 'OS=android target_arch=ia32', }" > chromium.gyp_env
or echo "{ 'GYP_DEFINES': 'OS=android target_arch=arm', }" > chromium.gyp_env
$ xwalk/build/android/envsetup.sh
$ export GYP_GENERATORS='ninja'
$ python xwalk/gyp_xwalk
$ ninja -C out/Release xwalk_core_shell_apk xwalk_runtime_client_shell_apk
$ ninja -C out/Release xwalk_runtime_lib_apk
```


# References
* https://crosswalk-project.org/contribute/building_crosswalk.html
