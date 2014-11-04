# Running a GPU process

```c++
void BrowserGpuChannelHostFactory::EstablishRequest::EstablishOnIO() {
  GpuProcessHost* host = GpuProcessHost::FromID(gpu_host_id_);
  if (!host) {
    host = GpuProcessHost::Get(GpuProcessHost::GPU_PROCESS_KIND_SANDBOXED,
                               cause_for_gpu_launch_);
    if (!host) {
      LOG(ERROR) << "Failed to launch GPU process.";
      FinishOnIO();
      return;
    }
    gpu_host_id_ = host->host_id();
    reused_gpu_process_ = false;
  } else {
```

```c++
GpuProcessHost::GpuProcessHost(int host_id, GpuProcessKind kind)
    : host_id_(host_id),
      valid_(true),
      in_process_(false),
      swiftshader_rendering_(false),
      kind_(kind),
      process_launched_(false),
      initialized_(false),
      gpu_crash_recorded_(false),
      uma_memory_stats_received_(false) {
  if (base::CommandLine::ForCurrentProcess()->HasSwitch(
          switches::kSingleProcess) ||
      base::CommandLine::ForCurrentProcess()->HasSwitch(
          switches::kInProcessGPU)) {
    in_process_ = true;
  }

  // If the 'single GPU process' policy ever changes, we still want to maintain
  // it for 'gpu thread' mode and only create one instance of host and thread.
  DCHECK(!in_process_ || g_gpu_process_hosts[kind] == NULL);

  g_gpu_process_hosts[kind] = this;

  // Post a task to create the corresponding GpuProcessHostUIShim.  The
  // GpuProcessHostUIShim will be destroyed if either the browser exits,
  // in which case it calls GpuProcessHostUIShim::DestroyAll, or the
  // GpuProcessHost is destroyed, which happens when the corresponding GPU
  // process terminates or fails to launch.
  BrowserThread::PostTask(
      BrowserThread::UI,
      FROM_HERE,
      base::Bind(base::IgnoreResult(&GpuProcessHostUIShim::Create), host_id));

  process_.reset(new BrowserChildProcessHostImpl(PROCESS_TYPE_GPU, this));
}
```

# Creating a ContextProviderCommandBuffer

```c++
ContextProviderCommandBuffer::Create(
    scoped_ptr<WebGraphicsContext3DCommandBufferImpl> context3d,
    const std::string& debug_name) {
  if (!context3d)
    return NULL;

  return new ContextProviderCommandBuffer(context3d.Pass(), debug_name);
}
```

# Here is the method that returns false
```C++
scoped_ptr<WebGraphicsContext3DCommandBufferImpl>
GpuProcessTransportFactory::CreateContextCommon(int surface_id) {
  if (!GpuDataManagerImpl::GetInstance()->CanUseGpuBrowserCompositor())
    return scoped_ptr<WebGraphicsContext3DCommandBufferImpl>();
  blink::WebGraphicsContext3D::Attributes attrs;
  attrs.shareResources = true;
  attrs.depth = false;
  attrs.stencil = false;
  attrs.antialias = false;
  attrs.noAutomaticFlushes = true;
  bool lose_context_when_out_of_memory = true;
  CauseForGpuLaunch cause =
      CAUSE_FOR_GPU_LAUNCH_WEBGRAPHICSCONTEXT3DCOMMANDBUFFERIMPL_INITIALIZE;
  scoped_refptr<GpuChannelHost> gpu_channel_host(
      BrowserGpuChannelHostFactory::instance()->EstablishGpuChannelSync(cause));
  if (!gpu_channel_host.get()) {
    LOG(ERROR) << "Failed to establish GPU channel.";
    return scoped_ptr<WebGraphicsContext3DCommandBufferImpl>();
  }
  GURL url("chrome://gpu/GpuProcessTransportFactory::CreateContextCommon");
  scoped_ptr<WebGraphicsContext3DCommandBufferImpl> context(
      new WebGraphicsContext3DCommandBufferImpl(
          surface_id,
          url,
          gpu_channel_host.get(),
          attrs,
          lose_context_when_out_of_memory,
          WebGraphicsContext3DCommandBufferImpl::SharedMemoryLimits(),
          NULL));
  return context.Pass();

```
# Creating a surface
```c++
scoped_ptr<cc::OverlayCandidateValidator> CreateOverlayCandidateValidator(
    gfx::AcceleratedWidget widget) {
#if defined(USE_OZONE)
  ui::OverlayCandidatesOzone* overlay_candidates =
      ui::SurfaceFactoryOzone::GetInstance()->GetOverlayCandidates(widget);
  if (overlay_candidates &&
      base::CommandLine::ForCurrentProcess()->HasSwitch(
          switches::kEnableHardwareOverlays)) {
    return scoped_ptr<cc::OverlayCandidateValidator>(
        new OverlayCandidateValidatorOzone(widget, overlay_candidates));
  }
#endif
  return scoped_ptr<cc::OverlayCandidateValidator>();
}

scoped_ptr<cc::OutputSurface> GpuProcessTransportFactory::CreateOutputSurface(
    ui::Compositor* compositor, bool software_fallback) {
  PerCompositorData* data = per_compositor_data_[compositor];
  if (!data)
    data = CreatePerCompositorData(compositor);

  bool create_software_renderer = software_fallback;
#if defined(OS_CHROMEOS)
  // Software fallback does not happen on Chrome OS.
  create_software_renderer = false;
#elif defined(OS_WIN)
  if (::GetProp(compositor->widget(), kForceSoftwareCompositor)) {
    if (::RemoveProp(compositor->widget(), kForceSoftwareCompositor))
      create_software_renderer = true;
  }
#endif

  scoped_refptr<ContextProviderCommandBuffer> context_provider;

  if (!create_software_renderer) {
    context_provider = ContextProviderCommandBuffer::Create(
        GpuProcessTransportFactory::CreateContextCommon(data->surface_id),
        "Compositor");
  }

  UMA_HISTOGRAM_BOOLEAN("Aura.CreatedGpuBrowserCompositor",
                        !!context_provider.get());

  if (context_provider.get()) {
    scoped_refptr<base::SingleThreadTaskRunner> compositor_thread_task_runner =
        GetCompositorMessageLoop();
    if (!compositor_thread_task_runner.get())
      compositor_thread_task_runner = base::MessageLoopProxy::current();

    // Here we know the GpuProcessHost has been set up, because we created a
    // context.
    output_surface_proxy_->ConnectToGpuProcessHost(
        compositor_thread_task_runner.get());
  }

  if (UseSurfacesEnabled()) {
    // This gets a bit confusing. Here we have a ContextProvider configured to
    // render directly to this widget. We need to make an OnscreenDisplayClient
    // associated with this context, then return a SurfaceDisplayOutputSurface
    // set up to draw to the display's surface.
    cc::SurfaceManager* manager = surface_manager_.get();
    scoped_ptr<cc::OutputSurface> display_surface;
    if (!context_provider.get()) {
      display_surface =
          make_scoped_ptr(new SoftwareBrowserCompositorOutputSurface(
              output_surface_proxy_,
              CreateSoftwareOutputDevice(compositor),
              per_compositor_data_[compositor]->surface_id,
              &output_surface_map_,
              compositor->vsync_manager()));
    } else {
      display_surface = make_scoped_ptr(new GpuBrowserCompositorOutputSurface(
          context_provider,
          per_compositor_data_[compositor]->surface_id,
          &output_surface_map_,
          compositor->vsync_manager(),
          CreateOverlayCandidateValidator(compositor->widget())));
    }
    scoped_ptr<OnscreenDisplayClient> display_client(new OnscreenDisplayClient(
        display_surface.Pass(), manager, compositor->task_runner()));

    scoped_refptr<cc::ContextProvider> offscreen_context_provider;
    if (context_provider.get()) {
      offscreen_context_provider = ContextProviderCommandBuffer::Create(
          GpuProcessTransportFactory::CreateOffscreenCommandBufferContext(),
          "Offscreen-Compositor");
    }
    scoped_ptr<SurfaceDisplayOutputSurface> output_surface(
        new SurfaceDisplayOutputSurface(manager,
          next_surface_id_namespace_++,
          offscreen_context_provider));
    display_client->set_surface_output_surface(output_surface.get());
    output_surface->set_display(display_client->display());
    data->display_client = display_client.Pass();
    return output_surface.PassAs<cc::OutputSurface>();
  }

  if (!context_provider.get()) {
    if (compositor_thread_.get()) {
      LOG(FATAL) << "Failed to create UI context, but can't use software"
                 " compositing with browser threaded compositing. Aborting.";
    }

    scoped_ptr<SoftwareBrowserCompositorOutputSurface> surface(
        new SoftwareBrowserCompositorOutputSurface(
            output_surface_proxy_,
            CreateSoftwareOutputDevice(compositor),
            per_compositor_data_[compositor]->surface_id,
            &output_surface_map_,
            compositor->vsync_manager()));
    return surface.PassAs<cc::OutputSurface>();
  }

  scoped_ptr<BrowserCompositorOutputSurface> surface;
#if defined(USE_OZONE)
  if (ui::SurfaceFactoryOzone::GetInstance()->CanShowPrimaryPlaneAsOverlay()) {
    surface.reset(new GpuSurfacelessBrowserCompositorOutputSurface(
        context_provider,
        per_compositor_data_[compositor]->surface_id,
        &output_surface_map_,
        compositor->vsync_manager(),
        CreateOverlayCandidateValidator(compositor->widget()),
        GL_RGB8_OES));
  }
#endif
  if (!surface)
    surface.reset(new GpuBrowserCompositorOutputSurface(
        context_provider,
        per_compositor_data_[compositor]->surface_id,
        &output_surface_map_,
        compositor->vsync_manager(),
        CreateOverlayCandidateValidator(compositor->widget())));

  if (data->reflector.get())
    data->reflector->ReattachToOutputSurfaceFromMainThread(surface.get());

  return surface.PassAs<cc::OutputSurface>();
}

scoped_refptr<ui::Reflector> GpuProcessTransportFactory::CreateReflector(
    ui::Compositor* source,
    ui::Layer* target) {
  PerCompositorData* data = per_compositor_data_[source];
  DCHECK(data);

  data->reflector = new ReflectorImpl(source,
                                      target,
                                      &output_surface_map_,
                                      GetCompositorMessageLoop(),
                                      data->surface_id);
  return data->reflector;
}

```
