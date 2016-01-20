# Run Crosswalk in Windows

```
 c:\git\crosswalk\src\out\Release\xwalk.exe manifest.json  >out.txt 2>err.txt
```

# Set fullscreen
```    
void FullscreenHandler::SetFullscreenImpl(bool fullscreen, bool for_metro) {
  ScopedFullscreenVisibility visibility(hwnd_);

  // Save current window state if not already fullscreen.
  if (!fullscreen_) {
    // Save current window information.  We force the window into restored mode
    // before going fullscreen because Windows doesn't seem to hide the
    // taskbar if the window is in the maximized state.
    saved_window_info_.maximized = !!::IsZoomed(hwnd_);
    if (saved_window_info_.maximized)
      ::SendMessage(hwnd_, WM_SYSCOMMAND, SC_RESTORE, 0);
    saved_window_info_.style = GetWindowLong(hwnd_, GWL_STYLE);
    saved_window_info_.ex_style = GetWindowLong(hwnd_, GWL_EXSTYLE);
    GetWindowRect(hwnd_, &saved_window_info_.window_rect);
  }

  fullscreen_ = fullscreen;
```
I tried to change the window style, but it doesn't.
It looks like the window style is overrided in other methods.
```
  if (fullscreen_) {
    // Set new window style and size.
    SetWindowLong(hwnd_, GWL_STYLE,
                  saved_window_info_.style & ~(WS_CAPTION | WS_THICKFRAME));
    SetWindowLong(hwnd_, GWL_EXSTYLE,
                  saved_window_info_.ex_style & ~(WS_EX_DLGMODALFRAME |
                  WS_EX_WINDOWEDGE | WS_EX_CLIENTEDGE | WS_EX_STATICEDGE));

    // On expand, if we're given a window_rect, grow to it, otherwise do
    // not resize.
    if (!for_metro) {
      MONITORINFO monitor_info;
      monitor_info.cbSize = sizeof(monitor_info);
      GetMonitorInfo(MonitorFromWindow(hwnd_, MONITOR_DEFAULTTONEAREST),
                     &monitor_info);
      gfx::Rect window_rect(monitor_info.rcMonitor);
      SetWindowPos(hwnd_, NULL, window_rect.x(), window_rect.y(),
                   window_rect.width(), window_rect.height(),
                   SWP_NOZORDER | SWP_NOACTIVATE | SWP_FRAMECHANGED);
    }
  } else {
    // Reset original window style and size.  The multiple window size/moves
    // here are ugly, but if SetWindowPos() doesn't redraw, the taskbar won't be
    // repainted.  Better-looking methods welcome.
    SetWindowLong(hwnd_, GWL_STYLE, saved_window_info_.style);
    SetWindowLong(hwnd_, GWL_EXSTYLE, saved_window_info_.ex_style);

    if (!for_metro) {
      // On restore, resize to the previous saved rect size.
      gfx::Rect new_rect(saved_window_info_.window_rect);
      SetWindowPos(hwnd_, NULL, new_rect.x(), new_rect.y(),
                   new_rect.width(), new_rect.height(),
                   SWP_NOZORDER | SWP_NOACTIVATE | SWP_FRAMECHANGED);
    }
    if (saved_window_info_.maximized)
      ::SendMessage(hwnd_, WM_SYSCOMMAND, SC_MAXIMIZE, 0);
  }
}
```

# Create a Window
```
WindowImpl::Init(HWND parent, const Rect& bounds)
HWDMessageHandler::Init
DesktopWindowTreeHostWin::Init
void DesktopNativeWidgetAura::InitNativeWidget
```

# Configuring the window style
DesktopWindowTreeHostWin::Init -> ConfigureWindowStyles

```
void ConfigureWindowStyles(
    HWNDMessageHandler* handler,
    const Widget::InitParams& params,
    WidgetDelegate* widget_delegate,
    internal::NativeWidgetDelegate* native_widget_delegate) {
  // Configure the HWNDMessageHandler with the appropriate
  DWORD style = 0;
  DWORD ex_style = 0;
  DWORD class_style = 0;
  CalculateWindowStylesFromInitParams(params, widget_delegate,
                                      native_widget_delegate, &style, &ex_style,
                                      &class_style);
  handler->set_initial_class_style(class_style);
  handler->set_window_style(handler->window_style() | style);
  handler->set_window_ex_style(handler->window_ex_style() | ex_style);
}
```
ui/views/widget/widget_hwnd_utils.cc

# Enable Metro UI in Chromium
```
#if defined(OS_WIN)
void BrowserView::SetMetroSnapMode(bool enable) {
  LOCAL_HISTOGRAM_COUNTS("Metro.SnapModeToggle", enable);
  ProcessFullscreen(enable, METRO_SNAP_FULLSCREEN, GURL(),
                    EXCLUSIVE_ACCESS_BUBBLE_TYPE_NONE);
}
```

# Stack Trace
```
base::debug::StackTrace::StackTrace [0x012B49F7+23] (c:\git\crosswalk\src\base\debug\stack_trace_win.cc:214)
views::FullscreenHandler::SetFullscreenImpl [0x02B7B3FC+44] (c:\git\crosswalk\src\ui\views\win\fullscreen_handler.cc:53)
views::FullscreenHandler::SetFullscreen [0x02B7B3C3+19] (c:\git\crosswalk\src\ui\views\win\fullscreen_handler.cc:32)
views::HWNDMessageHandler::SetFullscreen [0x02B6F16D+45] (c:\git\crosswalk\src\ui\views\win\hwnd_message_handler.cc:841)
views::DesktopWindowTreeHostWin::SetFullscreen [0x02B58D0F+31] (c:\git\crosswalk\src\ui\views\widget\desktop_aura\desktop_window_tree_host_win.cc:424)
views::Widget::SetFullscreen [0x02B43966+38] (c:\git\crosswalk\src\ui\views\widget\widget.cc:712)
xwalk::NativeAppWindowViews::SetFullscreen [0x012649B1+49] (c:\git\crosswalk\src\xwalk\runtime\browser\ui\native_app_window_views.cc:149)
xwalk::NativeAppWindowViews::Initialize [0x012648C1+433] (c:\git\crosswalk\src\xwalk\runtime\browser\ui\native_app_window_views.cc:88)
xwalk::NativeAppWindow::Create [0x01264398+40] (c:\git\crosswalk\src\xwalk\runtime\browser\ui\native_app_window_views.cc:333)
xwalk::DefaultRuntimeUIDelegate::Show [0x012653DB+59] (c:\git\crosswalk\src\xwalk\runtime\browser\runtime_ui_delegate.cc:73)
xwalk::application::ApplicationService::Launch [0x02B047B5+229] (c:\git\crosswalk\src\xwalk\application\browser\application_service.cc:52)
xwalk::application::ApplicationService::LaunchFromManifestPath [0x02B04A73+467] (c:\git\crosswalk\src\xwalk\application\browser\application_service.cc:86)
xwalk::application::ApplicationSystem::LaunchFromCommandLine [0x02B05BBA+218] (c:\git\crosswalk\src\xwalk\application\browser\application_system.cc:66)
xwalk::XWalkBrowserMainParts::PreMainMessageLoopRun [0x0125FF33+355] (c:\git\crosswalk\src\xwalk\runtime\browser\xwalk_browser_main_parts.cc:241)
content::BrowserMainLoop::PreMainMessageLoopRun [0x01539795+165] (c:\git\crosswalk\src\content\browser\browser_main_loop.cc:873)
content::StartupTaskRunner::RunAllTasksNow [0x01627A60+32] (c:\git\crosswalk\src\content\browser\startup_task_runner.cc:45)
content::BrowserMainLoop::CreateStartupTasks [0x01537521+625] (c:\git\crosswalk\src\content\browser\browser_main_loop.cc:763)
content::BrowserMainRunnerImpl::Initialize [0x01599180+608] (c:\git\crosswalk\src\content\browser\browser_main_runner.cc:206)
content::BrowserMain [0x030F3CAE+142] (c:\git\crosswalk\src\content\browser\browser_main.cc:22)
content::RunNamedProcessTypeMain [0x02B03D2E+286] (c:\git\crosswalk\src\content\app\content_main_runner.cc:379)
content::ContentMainRunnerImpl::Run [0x02B03BDD+125] (c:\git\crosswalk\src\content\app\content_main_runner.cc:804)
content::ContentMain [0x02B03263+35] (c:\git\crosswalk\src\content\app\content_main.cc:19)
wWinMain [0x0124AC7B+139] (c:\git\crosswalk\src\xwalk\runtime\app\xwalk_main.cc:85)
__tmainCRTStartup [0x02B8BBFB+253] (f:\dd\vctools\crt\crtw32\startup\crt0.c:251)
BaseThreadInitThunk [0x74E97C04+36]
RtlInitializeExceptionChain [0x776CAD5F+143]
RtlInitializeExceptionChain [0x776CAD2A+90]
```
```
base::debug::StackTrace::StackTrace [0x012B49F7+23] (c:\git\crosswalk\src\base\debug\stack_trace_win.cc:214)
	views::HWNDMessageHandler::ShowWindowWithState [0x02B6F548+216] (c:\git\crosswalk\src\ui\views\win\hwnd_message_handler.cc:592)
	views::DesktopWindowTreeHostWin::ShowWindowWithState [0x02B591EF+31] (c:\git\crosswalk\src\ui\views\widget\desktop_aura\desktop_window_tree_host_win.cc:211)
	views::DesktopNativeWidgetAura::ShowWithWindowState [0x02B46727+23] (c:\git\crosswalk\src\ui\views\widget\desktop_aura\desktop_native_widget_aura.cc:767)
	views::Widget::Show [0x02B43CD7+407] (c:\git\crosswalk\src\ui\views\widget\widget.cc:633)
	xwalk::application::Application::Launch [0x02B08DD0+608] (c:\git\crosswalk\src\xwalk\application\browser\application.cc:244)
	xwalk::application::ApplicationService::Launch [0x02B047B5+229] (c:\git\crosswalk\src\xwalk\application\browser\application_service.cc:52)
	xwalk::application::ApplicationService::LaunchFromManifestPath [0x02B04A73+467] (c:\git\crosswalk\src\xwalk\application\browser\application_service.cc:86)
	xwalk::application::ApplicationSystem::LaunchFromCommandLine [0x02B05BBA+218] (c:\git\crosswalk\src\xwalk\application\browser\application_system.cc:66)
	xwalk::XWalkBrowserMainParts::PreMainMessageLoopRun [0x0125FF33+355] (c:\git\crosswalk\src\xwalk\runtime\browser\xwalk_browser_main_parts.cc:241)
	content::BrowserMainLoop::PreMainMessageLoopRun [0x01539795+165] (c:\git\crosswalk\src\content\browser\browser_main_loop.cc:873)
	content::StartupTaskRunner::RunAllTasksNow [0x01627A60+32] (c:\git\crosswalk\src\content\browser\startup_task_runner.cc:45)
	content::BrowserMainLoop::CreateStartupTasks [0x01537521+625] (c:\git\crosswalk\src\content\browser\browser_main_loop.cc:763)
	content::BrowserMainRunnerImpl::Initialize [0x01599180+608] (c:\git\crosswalk\src\content\browser\browser_main_runner.cc:206)
	content::BrowserMain [0x030F3CAE+142] (c:\git\crosswalk\src\content\browser\browser_main.cc:22)
	content::RunNamedProcessTypeMain [0x02B03D2E+286] (c:\git\crosswalk\src\content\app\content_main_runner.cc:379)
	content::ContentMainRunnerImpl::Run [0x02B03BDD+125] (c:\git\crosswalk\src\content\app\content_main_runner.cc:804)
	content::ContentMain [0x02B03263+35] (c:\git\crosswalk\src\content\app\content_main.cc:19)
	wWinMain [0x0124AC7B+139] (c:\git\crosswalk\src\xwalk\runtime\app\xwalk_main.cc:85)
	__tmainCRTStartup [0x02B8BBFB+253] (f:\dd\vctools\crt\crtw32\startup\crt0.c:251)
	BaseThreadInitThunk [0x74E97C04+36]
	RtlInitializeExceptionChain [0x776CAD5F+143]
	RtlInitializeExceptionChain [0x776CAD2A+90]
	```

# Launch

```
bool Application::Launch() {
...
  runtime->set_ui_delegate(RuntimeUIDelegate::Create(runtime, params));
  runtime->Show();
...
}
```
↓
```
RuntimeUIDelegate* RuntimeUIDelegate::Create(
    Runtime* runtime,
    const NativeAppWindow::CreateParams& params) {
#if defined(OS_WIN) || defined(OS_LINUX)
  return new RuntimeUIDelegateDesktop(runtime, params);
#else
  return new DefaultRuntimeUIDelegate(runtime, params);
#endif
}
```
↓
```
RuntimeUIDelegateDesktop::RuntimeUIDelegateDesktop(
    Runtime* runtime, const NativeAppWindow::CreateParams& params)
  : DefaultRuntimeUIDelegate(runtime, params) {
}

```
↓

```
DefaultRuntimeUIDelegate::DefaultRuntimeUIDelegate(
    Runtime* runtime, const NativeAppWindow::CreateParams& params)
  : runtime_(runtime),
    window_(nullptr),
    window_params_(params) {
  DCHECK(runtime_);
}
```
