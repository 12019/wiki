# ozone-wayalnd



## Show(), Hide()

```
void DesktopWindowTreeHostOzone::Show() {
  LOG(INFO) << "DesktopWindowTreeHostOzone::" <<  __func__;
  if (state_ & Visible)
    return;

  platform_window_->Show();
  ShowWindow();
  native_widget_delegate_->OnNativeWidgetVisibilityChanged(true);
}

void DesktopWindowTreeHostOzone::Hide() {
  LOG(INFO) << "DesktopWindowTreeHostOzone::" <<  __func__;
  if (!(state_ & Visible))
    return;

  state_ &= ~Visible;
  platform_window_->Hide();
  native_widget_delegate_->OnNativeWidgetVisibilityChanged(false);
}
```
ozone/ui/desktop_aura/desktop_window_tree_host_ozone.cc


# ozone

* https://codereview.chromium.org/1610683003
* https://chromium.googlesource.com/chromium/src.git/+/master/ui/ozone/platform/wayland/
