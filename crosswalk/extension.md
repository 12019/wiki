

## Create an extension process from the runtime process(browser process)
```
63│ void XWalkAppExtensionBridge::ExtensionProcessCreated(
64│     int render_process_id,
65│     const IPC::ChannelHandle& channel_handle) {
66│ #if defined(OS_LINUX)
67│   CHECK(app_system_);
68│   application::ApplicationService* service = app_system_->application_service();
69│   application::Application* app =
70│       service->GetApplicationByRenderHostID(render_process_id);
71├>  CHECK(app);
72│
73│   application::ApplicationSystemLinux* app_system =
74│       static_cast<application::ApplicationSystemLinux*>(app_system_);
75│   application::RunningApplicationObject* running_app_object =
76│       app_system->service_provider()->GetRunningApplicationObject(app);
77│   CHECK(running_app_object);
78│   running_app_object->ExtensionProcessCreated(channel_handle);
79│ #endif
80│ }
```
xwalk/runtime/browser/xwalk_app_extension_bridge.cc
