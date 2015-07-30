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
$ adb -s emulator-5554 install -r out/Release/apks/XWalkRuntimeLib.apk
$ adb -s emulator-5554 install -r out/Release/apks/XWalkRuntimeClientShell.apk

```
Running a MinBrowser (without loading a page)
```
$ adb shell am start -n org.xwalk.runtime.client.shell/org.xwalk.runtime.client.shell.XWalkRuntimeClientShellActivity
```

### Creating a package
https://crosswalk-project.org/documentation/android/android_target_setup.html
#### Tips
```
$ adb devices -l
$ adb logcat
$ adb push anand.jpg /data/local
```
# References
* https://crosswalk-project.org/contribute/building_crosswalk.html
* https://github.com/crosswalk-project/crosswalk-website/wiki/Android-debugging
* http://developer.android.com/intl/es/tools/publishing/app-signing.html
