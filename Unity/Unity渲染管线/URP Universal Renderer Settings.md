# [[渲染路径]]
URP Universal Renderer主要实现两种渲染路径：[[Forward Rendering Path]]和[[Deferred Rendering Path]]。
而Forward Rendering Path这一个渲染路径里面还有衍生的[[Forward+ Rendering Path]].

## 渲染路径对比

The following table shows the differences between the Forward and the Deferred Rendering Paths in URP.

| Feature                                        | Forward                               | Forward+                                                                                                                                                               | Deferred                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ---------------------------------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Maximum number of real-time lights per object. | 9                                     | Unlimited. [The per-Camera limit applies](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/rendering/forward-plus-rendering-path.html). | Unlimited                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Per-pixel normal encoding                      | No encoding (accurate normal values). | No encoding (accurate normal values).                                                                                                                                  | Two options:<br><br>- Quantization of normals in G-buffer (loss of accuracy, better performance).<br>- Octahedron encoding (accurate normals, might have significant performance impact on mobile GPUs).<br><br>For more information, see the section [Encoding of normals in G-buffer](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/rendering/deferred-rendering-path.html#accurate-g-buffer-normals). |
| MSAA                                           | Yes                                   | Yes                                                                                                                                                                    | No                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Vertex lighting                                | Yes                                   | No                                                                                                                                                                     | No                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Camera stacking                                | Yes                                   | Yes                                                                                                                                                                    | Supported with a limitation: Unity renders only the base Camera using the Deferred Rendering Path. Unity renders all overlay Cameras using the Forward Rendering Path.                                                                                                                                                                                                                                                                     |
## Universal Renderer asset reference

This section describes the properties of the Forward Renderer asset.

![URP Universal Renderer](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/images/urp-assets/urp-universal-renderer.png)

### Filtering

选择render绘制的图层，分成不透明物体和透明物体两种。

|Property|Description|
|---|---|
|**Opaque Layer Mask**|Select which opaque layers this Renderer draws|
|**Transparent Layer Mask**|Select which transparent layers this Renderer draws|

### Rendering

This section contains properties related to rendering.

| Property                      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Rendering Path**            | Select the Rendering Path.  <br>Options:<br><br>- **Forward**: The Forward Rendering Path.<br>- **Forward+**: The [Forward+ Rendering Path](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/rendering/forward-plus-rendering-path.html).<br>- **Deferred**: The [Deferred Rendering Path](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/rendering/deferred-rendering-path.html).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| **Depth Priming Mode**        | This property determines when Unity performs depth priming.  <br>Depth Priming can improve GPU frame timings by reducing the number of pixel shader executions. The performance improvement depends on the amount of overlapping pixels in the opaque pass and the complexity of the pixel shaders that Unity can skip by using depth priming.  <br>The feature has an upfront memory and performance cost. The feature uses a depth prepass to determine which pixel shader invocations Unity can skip, and the feature adds the depth prepass if it's not available yet.  <br>The options are:<br><br>- **Disabled**: Unity does not perform depth priming.<br>- **Auto**: If there is a Render Pass that requires a depth prepass, Unity performs the depth prepass and depth priming.<br>- **Forced**: Unity always performs depth priming. To do this, Unity also performs a depth prepass for every render pass. **Note**: Depth priming is disabled at runtime on certain hardware (Tile Based Deferred Rendering) regardless of this setting.<br><br>On Android, iOS, and Apple TV, Unity performs depth priming only in the Forced mode. On tiled GPUs, which are common to those platforms, depth priming might reduce performance when combined with MSAA.  <br>  <br>This property is available only if **Rendering Path** is set to **Forward** |
| **Accurate G-buffer normals** | Indicates whether to use a more resource-intensive normal encoding/decoding method to improve visual quality.  <br>仅仅在延迟渲染有效果。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| **Copy Depth Mode**           | Specifies the stage in the render pipeline at which to copy the scene depth to a depth texture. The options are:<br><br>- **After Opaques**: URP copies the scene depth after the opaques render pass.<br>- **After Transparents**: URP copies the scene depth after the transparents render pass.<br>- **Force Prepass**: URP does a depth prepass to generate the scene depth texture.<br><br>**Note**: On mobile devices, the **After Transparents** option can lead to a significant improvement in memory bandwidth. This is because Copy Depth causes a switch in render target between the Opaque pass and the Transparents pass. When this occurs, Unity stores the contents of the Color Buffer in main memory then loads it again once the Copy Depth is complete. The impact increases significantly when MSAA is enabled as Unity must also store and load the MSAA data alongside the Color Buffer.                                                                                                                                                                                                                                                                                                                                                                                                                                             |

### Native RenderPass

This section contains properties related to URP's Native RenderPass API.

|Property|Description|
|---|---|
|**Native RenderPass**|Indicates whether to use URP's Native RenderPass API. When enabled, URP uses this API to structure render passes. As a result, you can use [programmable blending](https://docs.unity.cn/Manual/SL-PlatformDifferences.html#using-shader-framebuffer-fetch) in custom URP shaders. For more information about the RenderPass API, see [ScriptableRenderContext.BeginRenderPass](https://docs.unity.cn/ScriptReference/Rendering.ScriptableRenderContext.BeginRenderPass.html).  <br>  <br>**Note**: Enabling this property has no effect on OpenGL ES.|

### Shadows

This section contains properties related to rendering shadows.

|Property|Description|
|---|---|
|**Transparent Receive Shadows**|When this option is on, Unity draws shadows on transparent objects.|

### Overrides

This section contains Render Pipeline properties that this Renderer overrides.

#### Stencil

With this check box selected, the Renderer processes the Stencil buffer values.

![URP Universal Renderer Stencil override](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/images/urp-assets/urp-universal-renderer-stencil-on.png)

For more information on how Unity works with the Stencil buffer, see [ShaderLab: Stencil](https://docs.unity.cn/Manual/SL-Stencil.html).

### Compatibility

This section contains settings related to backwards compatibility.

|Property|Description|
|---|---|
|**Intermediate Texture**|This property lets you force URP to renders via an intermediate texture.  <br>Options:<br><br>- **Auto**: URP uses the information provided by the `ScriptableRenderPass.ConfigureInput` method to determine automatically whether rendering via an intermediate texture is necessary.<br>- **Always**: forces rendering via an intermediate texture. Use this option only for compatibility with Renderer Features that do not declare their inputs with `ScriptableRenderPass.ConfigureInput`. Using this option might have a significant performance impact on some platforms.|

### Renderer Features

This section contains the list of Renderer Features assigned to the selected Renderer.

For information on how to add a Renderer Feature, see [How to add a Renderer Feature to a Renderer](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/urp-renderer-feature-how-to-add.html).

URP contains the pre-built Renderer Feature called [Render Objects](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/renderer-features/renderer-feature-render-objects.html).