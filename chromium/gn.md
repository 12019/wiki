
## Debugging GN
```
diff --git a/build/config/BUILD.gn b/build/config/BUILD.gn
index 32ec333..a688458 100644
--- a/build/config/BUILD.gn
+++ b/build/config/BUILD.gn
@@ -124,6 +124,7 @@ config("feature_flags") {
   }   
   if (use_cairo) {
     defines += [ "USE_CAIRO=1" ]
+    print("USE_CAIRO=1")
   }   
   if (use_clipboard_aurax11) {
     defines += [ "USE_CLIPBOARD_AURAX11=1" ]
diff --git a/build/config/linux/BUILD.gn b/build/config/linux/BUILD.gn
index c0871f8..afbf766 100644
--- a/build/config/linux/BUILD.gn
+++ b/build/config/linux/BUILD.gn
@@ -53,6 +53,9 @@ if (use_pango || use_cairo) {
     packages = [ "pangocairo" ]
   }   
 }
+  pkg_config("cairo") {
+    packages = [ "cairo" ]
+  }
 
 if (use_pango) {
   pkg_config("pangoft2") {
diff --git a/build/config/ui.gni b/build/config/ui.gni
index a102563..8d23791 100644
--- a/build/config/ui.gni
+++ b/build/config/ui.gni
@@ -81,6 +81,8 @@ if (is_linux && !use_ozone) {
   use_pango = false
 }
 
+print(use_cairo)
+
 # Whether to use atk, the Accessibility ToolKit library
 use_atk = is_desktop_linux && use_x11
 
diff --git a/ui/gfx/BUILD.gn b/ui/gfx/BUILD.gn
index d014bd7..0e67552 100644
--- a/ui/gfx/BUILD.gn
+++ b/ui/gfx/BUILD.gn
@@ -434,6 +434,7 @@ component("gfx") {
 
   if (use_cairo) {
     configs += [ "//build/config/linux:pangocairo" ]
+    print("add pangocairo")
   }   
 }
 ```
