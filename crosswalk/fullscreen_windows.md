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
