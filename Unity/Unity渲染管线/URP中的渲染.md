The Universal Render Pipeline ([[URP]]) renders scenes using the following components:

- URP Renderer. URP contains the following Renderers:
    - [Universal Renderer](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/urp-universal-renderer.html).
    - [2D Renderer](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/Setup.html#2d-renderer-setup).
- [Shading models](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/shading-model.html) for shaders shipped with URP
- Camera
- [URP Asset](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/universalrp-asset.html)

The following illustration shows the frame rendering loop of the URP Universal Renderer.

![URP Universal Renderer, Forward Rendering Path](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/images/Graphics/Rendering_Flowchart.png)

When the [render pipeline is active in Graphics Settings](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/InstallURPIntoAProject.html), Unity uses URP to render all Cameras in your Project, including game and Scene view cameras, Reflection Probes, and the preview windows in your Inspectors.

The URP renderer executes a Camera loop for each Camera, which performs the following steps:

1. Culls rendered objects in your scene
2. Builds data for the renderer
3. Executes a renderer that outputs an image to the framebuffer.

For more information about each step, see [Camera loop](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/rendering-in-universalrp.html#camera-loop).

In the [RenderPipelineManager](https://docs.unity.cn/ScriptReference/Rendering.RenderPipelineManager.html) class, URP provides events that you can use to execute code before and after rendering a frame, and before and after rendering each Camera loop. The events are:

- [beginCameraRendering](https://docs.unity.cn/ScriptReference/Rendering.RenderPipelineManager-beginCameraRendering.html)
- [beginFrameRendering](https://docs.unity.cn/ScriptReference/Rendering.RenderPipelineManager-beginFrameRendering.html)
- [endCameraRendering](https://docs.unity.cn/ScriptReference/Rendering.RenderPipelineManager-endCameraRendering.html)
- [endFrameRendering](https://docs.unity.cn/ScriptReference/Rendering.RenderPipelineManager-endFrameRendering.html)

For the example of how to use the beginCameraRendering event, see the page [Using the beginCameraRendering event](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/using-begincamerarendering.html).

## Camera loop

The Camera loop performs the following steps:

|Step|Description|
|---|---|
|**Setup Culling Parameters**|Configures parameters that determine how the culling system culls Lights and shadows. You can override this part of the render pipeline with a custom renderer.|
|**Culling**|Uses the culling parameters from the previous step to compute a list of visible renderers, shadow casters, and Lights that are visible to the Camera. Culling parameters and Camera [layer distances](https://docs.unity.cn/ScriptReference/Camera-layerCullDistances.html) affect culling and rendering performance.|
|**Build Rendering Data**|Catches information based on the culling output, quality settings from the [URP Asset](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/universalrp-asset.html), [Camera](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/cameras.html), and the current running platform to build the `RenderingData`. The rendering data tells the renderer the amount of rendering work and quality required for the Camera and the currently chosen platform.|
|**Setup Renderer**|Builds a list of render passes, and queues them for execution according to the rendering data. You can override this part of the render pipeline with a custom renderer.|
|**Execute Renderer**|Executes each render pass in the queue. The renderer outputs the Camera image to the framebuffer.|
