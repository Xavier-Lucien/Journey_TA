# ShaderLab的组成：
下面是最基础的结构：
material properties是留给美术调整材质观感到底。
一般来说Subshader是针对不同显卡写的，好的显卡复杂一些，差劲的显卡也要写对应的算法。
fallback是保底。
`Shader "<name>"`  
`{`  
    `<optional: Material properties>`  
    `<One or more SubShader definitions>`  
    `<optional: custom editor>`  
    `<optional: fallback>`  
`}`

## 材质属性：
遵循这个格式
[optional: attribute] name("display text in Inspector", type name) = default value

按变量类型划分的材质属性：

| 类型           | 示例语法                                                                                                           | 注释                                                                                                                                                                                                                                       |
| ------------ | -------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 整数           | _ExampleName ("Integer display name", Integer) = 1                                                             |                                                                                                                                                                                                                                          |
| 浮点数          | _ExampleName ("Float display name", Float) = 0.5<br>_ExampleName ("Float with range", Range(0.0, 1.0)) = 0.5   |                                                                                                                                                                                                                                          |
| 纹理2D         | _ExampleName ("Texture2D display name", 2D) = "" {} <br>_ExampleName ("Texture2D display name", 2D) = "red" {} | 将以下值置于默认值字符串中可使用 Unity 的内置纹理之一：“white”（RGBA：1,1,1,1）、“black”（RGBA：0,0,0,1）、“gray”（RGBA：0.5,0.5,0.5,1）、“bump”（RGBA：0.5,0.5,1,0.5）或“red”（RGBA：1,0,0,1）。  <br>  <br>如果将该字符串留空或输入无效值，则它默认为 “gray”。  <br>  <br>**注意：**这些默认纹理在 Inspector 中不可见。 |
| 纹理2D数组       | _ExampleName ("Texture2DArray display name", 2DArray) = "" {}                                                  |                                                                                                                                                                                                                                          |
| 纹理3D         | _ExampleName ("Texture3D", 3D) = "" {}                                                                         |                                                                                                                                                                                                                                          |
| CubeMap      | _ExampleName ("Cubemap", Cube) = "" {}                                                                         |                                                                                                                                                                                                                                          |
| CubeMapArray | _ExampleName ("CubemapArray", CubeArray) = "" {}                                                               |                                                                                                                                                                                                                                          |
| Color        | _ExampleName("Example color", Color) = (.25, .5, .5, 1)                                                        | 这会在着色器代码中映射到 float4。  <br>  <br>材质 Inspector 会显示一个拾色器。如果更愿意将值作为四个单独的浮点数进行编辑，请使用 Vector 类型。                                                                                                                                               |
| Vector       | _ExampleName ("Example vector", Vector) = (.25, .5, .5, 1)                                                     | 这会在着色器代码中映射到 float4。  <br>  <br>材质 Inspector 会显示四个单独的浮点数字段。如果更愿意使用拾色器编辑值，请使用 Color 类型。                                                                                                                                                   |
材质属性特性：

| 属性                | 功能                                                                                                                                                                                                                                                                                                                                 |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Gamma]           |                                                                                                                                                                                                                                                                                                                                    |
| [HDR]             | 指示纹理或颜色属性使用[高动态范围 (HDR)](https://docs.unity.cn/cn/2022.3/Manual/HDR.html) 值。  <br>  <br>对于纹理属性，如果分配了 LDR 纹理，则 Unity 编辑器会显示警告。对于颜色属性，Unity 编辑器会使用 HDR 拾色器编辑此值。                                                                                                                                                                      |
| [HideInInspector] | 告知 Unity 编辑器在 Inspector 中隐藏此属性。                                                                                                                                                                                                                                                                                                    |
| [MainTexture]     | 为材质设置主纹理，可以使用 [Material.mainTexture](https://docs.unity.cn/cn/2022.3/ScriptReference/Material-mainTexture.html) 进行访问。  <br>  <br>默认情况下，Unity 将具有属性名称 `_MainTex` 的纹理视为主纹理。如果纹理具有不同的属性名称，但希望 Unity 将它视为主纹理，请使用此特性。  <br>  <br>如果多次使用此特性，则 Unity 会使用第一个属性并忽略后续属性。  <br>  <br>**注意：**使用此特性设置主纹理时，如果使用纹理串流调试视图模式或自定义调试工具，则该纹理在游戏视图中不可见。 |
| [MainColor]       | 为材质设置主色，可以使用 [Material.color](https://docs.unity.cn/cn/2022.3/ScriptReference/Material-color.html) 进行访问。  <br>  <br>默认情况下，Unity 将具有属性名称 `_Color` 的颜色视为主色。如果您的颜色具有其他属性 (property) 名称，但您希望 Unity 将这个颜色视为主色，请使用此属性 (attribute)。如果您多次使用此属性 (attribute)，则 Unity 会使用第一个属性 (property)，而忽略后续属性 (property)。                                 |
| [NoScaleOffset]   | 告知 Unity 编辑器隐藏此纹理属性的平铺和偏移字段。                                                                                                                                                                                                                                                                                                       |
| [Normal]          | 指示纹理属性需要法线贴图。  <br>  <br>如果分配了不兼容的纹理，则 Unity 编辑器会显示警告。                                                                                                                                                                                                                                                                             |
| [PerRendererData] | 指示纹理属性将来自每渲染器数据，形式为 [MaterialPropertyBlock](https://docs.unity.cn/cn/2022.3/ScriptReference/MaterialPropertyBlock.html)。  <br>  <br>材质 Inspector 会将这些属性显示为只读。                                                                                                                                                                      |
## 自定义编辑器：
这个好像不常用，后面再看看

## Subshader：
在 ShaderLab 中，通过将 `SubShader` 代码块置于 `Shader` 代码块中，可以定义子着色器。

在 `SubShader` 代码块中，可以：

- 使用 `LOD` 代码块为 SubShader 分配 LOD（细节级别）值。参阅[向 SubShader 分配 LOD 值](https://docs.unity.cn/cn/2022.3/Manual/SL-ShaderLOD.html)。
- 使用 `Tags` 代码块将数据的键值对分配给子着色器。参阅 [ShaderLab：向子着色器分配标签](https://docs.unity.cn/cn/2022.3/Manual/SL-SubShaderTags.html)。
- 使用 ShaderLab 命令将 GPU 指令或着色器代码添加到 SubShader。请参阅 [ShaderLab：使用命令](https://docs.unity.cn/cn/2022.3/Manual/shader-shaderlab-commands.html)。
- 使用 `Pass` 代码块定义一个或多个通道。参阅 [ShaderLab：定义通道](https://docs.unity.cn/cn/2022.3/Manual/SL-Pass.html)。
- Specify package requirements using the `PackageRequirements` block. This makes Unity only run the SubShader if the required packages are installed. See [ShaderLab: specifying package requirements](https://docs.unity.cn/cn/2022.3/Manual/SL-PackageRequirements.html).

| **结构**                                                                                                                                                         | **功能**                                 |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| `SubShader`  <br>`{`  <br>    `<optional: LOD>`  <br>    `<optional: tags>`  <br>    `<optional: commands>`  <br>    `<One or more Pass definitions>`  <br>`}` | 定义子着色器。  <br>  <br>您可以在子着色器中定义任意数量的通道。 |


```
Shader "Examples/SinglePass"
{
    SubShader
    {
        Tags { "ExampleSubShaderTagKey" = "ExampleSubShaderTagValue" }
        LOD 100

         // 此处是应用于整个子着色器的 ShaderLab 命令。

        Pass
        {                
              Name "ExamplePassName"
              Tags { "ExamplePassTagKey" = "ExamplePassTagValue" }

              // 此处是应用于此通道的 ShaderLab 命令。

              // 此处是 HLSL 代码。
        }
    }
}
```

### Tags代码块：
在 ShaderLab 中，可以通过将 `Tags` 代码块置于 `SubShader` 代码块中来向子着色器分配标签。

请注意，子着色器和通道都使用 `Tags` 代码块，但其工作方式不同。向通道分配子着色器标签没有效果，反之亦然。区别在于放置 `Tags` 代码块的位置：

- 要定义通道标签，请将 `Tags` 代码块置于 `Pass` 代码块内部。
- 要定义子着色器标签，请将 `Tags` 代码块置于 `SubShader` 代码块内部，但是在 `Pass` 代码块外部。
可以使用 [Material.GetTag](https://docs.unity.cn/cn/2022.3/ScriptReference/Material.GetTag.html) API 从 C# 脚本中读取子着色器标签，如下所示：

```
using UnityEngine;

public class Example : MonoBehaviour
{
    // 将此附加到具有渲染器组件的游戏对象
    string tagName = "ExampleTagName";

    void Start()
    {
        Renderer myRenderer = GetComponent<Renderer>();
        string tagValue = myRenderer.material.GetTag(ExampleTagName, true, "Tag not found");
        Debug.Log(tagValue);
    }
}
```
SubShader一共有这么多的Tags种类：

| key                    | value                                                                  | meaning                                                                                                                                                                                                                                                                                                                                                              |
| ---------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| “RenderPipeline"       | "UniversalRenderPipeline"<br>"HighDefinitionRenderPipeline"<br>其他值或不声明 | 此着色器仅与URP相容<br>此着色器仅与HDRP相容<br>与URP和HDRP都不兼容                                                                                                                                                                                                                                                                                                                         |
| ”Queue“                | ”[queue name]"<br>"[queue name]+[offset]"                              | 使用命名渲染队列<br>在队列区分好以后在进行顺序上面的微调，水渲染就是一共典型案例                                                                                                                                                                                                                                                                                                                           |
| “RenderType"           | string                                                                 | Unity没有为这个key设置固定的value与之匹配。                                                                                                                                                                                                                                                                                                                                         |
| "ForceNoShadowCasting" | "True"<br><br><br><br><br><br><br><br><br><br><br><br><br>"False"      | Unity prevents the geometry in this SubShader from casting shadows.  <br>In the Built in Render Pipeline, with the Forward or Legacy Vertex Lit rendering paths, Unity also prevents the geometry in this SubShader from receiving shadows.  <br>In HDRP, this does not prevent the geometry from casting contact shadows.<br><br>Unity 不会阻止此子着色器中的几何体投射或接收阴影。这是默认值。 |
| "DisableBatching"      | "True"<br><br>"False"<br><br>"LODFading"                               | Unity对使用此着色器的物体禁止动态批处理<br>Unity允许其进行动态批处理，这是默认值<br>对于属于 Fade Mode 值不为 None 的 [LODGroup](https://docs.unity.cn/Manual/class-LODGroup.html) 一部分的所有几何体，Unity 会阻止动态批处理。否则，Unity 不会阻止动态批处理。                                                                                                                                                                               |
| “IgnoreProjector"      | "True"<br><br>"False"                                                  | Unity 在渲染此几何体时忽略投影器。<br>Unity 在渲染此几何体时不会忽略投影器。这是默认值。                                                                                                                                                                                                                                                                                                                 |
| "PreviewType"          | "Sphere"<br>"Plane"<br>"Skybox"                                        | 在球体上显示材质，这是默认值<br>在平面上显示材质<br>在天空盒上显示材质                                                                                                                                                                                                                                                                                                                              |
| ”CanUseSpriteAtlas"    | <br><br><br><br><br>"True"<br><br><br>"False"                          | 在使用 [Legacy Sprite Packer](https://docs.unity.cn/cn/2022.3/Manual/SpritePacker.html) 的项目中使用此子着色器标签可警告用户着色器依赖于原始纹理坐标，因此不应将其纹理打包到图集中。<br><br>使用此子着色器的精灵与 Legacy Sprite Packer 兼容。这是默认值。<br><br>使用此子着色器的精灵与 Legacy Sprite Packer 不兼容。  当 `CanUseSpriteAtlas` 值为 `False` 的子着色器与带有 Legacy Sprite Packer 打包标签的精灵一起使用时，Unity 会在 Inspector 中显示错误消息。                          |

### LOD代码块：
Shader里面的LOD和模型的LOD完全是两回事。模型里面的LOD是用新建一共空的游戏物体，添加LOD Group组件，然后给LOD0，LOD1，LOD2这些模型放进去。
SubShader里面的LOD则是分配不同的光照算法，按照复杂度降序排列。LOD500，LOD200，LOD100等这样依次向下排列。
你可以在C#脚本里面对物理的着色方式进行控制，To set the shader LOD for a given Shader object, you can use [Shader.maximumLOD](https://docs.unity.cn/cn/2022.3/ScriptReference/Shader-maximumLOD.html). To set the shader LOD for all Shader objects, you can use [Shader.globalMaximumLOD](https://docs.unity.cn/cn/2022.3/ScriptReference/Shader-globalMaximumLOD.html). By default, there is no maximum LOD.

### Pass代码块:
这是最核心的代码块
Pass的结构：
`Pass`  
`{`  
    `<optional: name>`  
 `<optional: tags>`  
    `<optional: commands>`  
   `<optional: shader code>`  
`}`
#### name模块：
通道可以具有名称。您需要在 `UsePass` 命令及某些 C# API 中按名称引用一个通道。通道的名称在frame debugger[帧调试器](https://docs.unity.cn/cn/2022.3/Manual/FrameDebugger.html)工具中可见。
要在 ShaderLab 中为通道指定名称，您可以在 `Pass` 代码块内放置一个 `Name` 代码块。

| **Tag**         | **Function** |
| --------------- | ------------ |
| `Name "<name>"` | 设置通道的名称。     |

在内部，Unity 将名称转换为大写。在 ShaderLab 代码中引用名称时，必须使用大写变体；例如，如果值是 “example”，您必须使用 EXAMPLE 进行引用。
To access the name of a Pass from C# scripts, you can use APIs such as [Material.FindPass](https://docs.unity.cn/cn/2022.3/ScriptReference/Material.FindPass.html), [Material.GetPassName](https://docs.unity.cn/cn/2022.3/ScriptReference/Material.GetPassName.html), or [ShaderData.Pass.Name](https://docs.unity.cn/cn/2022.3/ScriptReference/ShaderData.Pass.Name.html).
**注意：**[Material.GetShaderPassEnabled](https://docs.unity.cn/cn/2022.3/ScriptReference/Material.GetShaderPassEnabled.html) 和 [Material.SetShaderPassEnabled](https://docs.unity.cn/cn/2022.3/ScriptReference/Material.SetShaderPassEnabled.html) 不按名称引用通道；而是使用 [LightMode 标签](https://docs.unity.cn/cn/2022.3/Manual/SL-PassTags.html)的值引用通道。
此示例代码创建了一个名为 ContainsNamedPass 的 Shader 对象，其中包含名为 ExampleNamedPass 的通道。

```
Shader "Examples/ContainsNamedPass"
{
    SubShader
    {
        Pass
        {    
              Name "ExampleNamedPass"
            
              // 此处是定义通道的代码的其余部分。
        }
    }
}
```

然后您可以使用以下 C# 代码查询此通道的名称：

```
using UnityEngine;

public class GetPassName : MonoBehaviour
{
    // 将此脚本放置在具有 MeshRenderer 组件的游戏对象上
    
    void Start() {
        // 获取材质
        var material = GetComponent<MeshRenderer>().material;

        // 获取为该材质分配的 Shader 对象的
        // 活动子着色器中第一个通道的名称
        var passName = material.GetPassName(0);

        // 将名称打印到控制台
        Debug.Log(passName);
    }
}
```
#### Tags模块（主要是LightMode）：
因为目前基本都是在用URP渲染管线，所以作者总结了一些URP渲染管线下的LightMode。
The value of this tag lets the pipeline determine which Pass to use when executing different parts of the Render Pipeline.
If you do not set the `LightMode` tag in a Pass, URP uses the `SRPDefaultUnlit` tag value for that Pass.
In URP, the `LightMode` tag can have the following values.

| **Property**             | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **UniversalForward**     | 前向渲染中渲染物体并且计算光照                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **UniversalGBuffer**     | 延迟渲染中仅仅完成把物体渲染到GBuffer里面的操作                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| **UniversalForwardOnly** | 和UniversalForward类似，但是可以对前向渲染和延迟渲染两种渲染路径同时起作用。<br>Use this value if a certain Pass must render objects with the Forward Rendering Path when URP is using the Deferred Rendering Path. For example, use this tag if URP renders a Scene using the Deferred Rendering Path and the Scene contains objects with shader data that does not fit the GBuffer, such as Clear Coat normals.  <br>If a shader must render in both the Forward and the Deferred Rendering Paths, declare two Passes with the `UniversalForward` and `UniversalGBuffer` tag values. If a shader must render using the Forward Rendering Path regardless of the Rendering Path that the URP Renderer uses, declare only a Pass with the `LightMode` tag set to `UniversalForwardOnly`. |
| **Universal2D**          | The Pass renders objects and evaluates 2D light contributions. URP uses this tag value in the 2D Renderer.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| **ShadowCaster**         | The Pass renders object depth from the perspective of lights into the Shadow map or a depth texture.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **DepthOnly**            | The Pass renders only depth information from the perspective of a Camera into a depth texture.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| **Meta**                 | Unity executes this Pass only when baking lightmaps in the Unity Editor. Unity strips this Pass from shaders when building a Player.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **SRPDefaultUnlit**      | Use this `LightMode` tag value to draw an extra Pass when rendering objects. Application example: draw an object outline. This tag value is valid for both the Forward and the Deferred Rendering Paths.  <br>URP uses this tag value as the default value when a Pass does not have a `LightMode` tag.                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

> **NOTE**: URP does not support the following LightMode tags: `Always`, `ForwardAdd`, `PrepassBase`, `PrepassFinal`, `Vertex`, `VertexLMRGBM`, `VertexLM`.

[Material.SetShaderPassEnabled](https://docs.unity.cn/cn/2022.3/ScriptReference/Material.SetShaderPassEnabled.html) and [ShaderTagId](https://docs.unity.cn/cn/2022.3/ScriptReference/Rendering.ShaderTagId.html) use the value of the `LightMode` tag to determine how Unity handles a given Pass.

在可编程渲染管线中，您可以为 `LightMode` 标签创建自定义值。然后，通过配置一个 [DrawingSettings](https://docs.unity.cn/cn/2022.3/ScriptReference/Rendering.DrawingSettings.html) 结构，您可以使用这些自定义值来确定在给定 [ScriptableRenderContext.DrawRenderers](https://docs.unity.cn/cn/2022.3/ScriptReference/Rendering.ScriptableRenderContext.DrawRenderers.html) 调用期间要绘制哪些通道。有关更多信息和代码示例，请参阅[在自定义可编程渲染管线中创建一个简单的渲染循环](https://docs.unity.cn/cn/2022.3/Manual/srp-creating-simple-render-loop.html)。

```
Shader "Examples/ExampleLightMode"
{
    SubShader
    {
        Pass
        {    
              Tags { "LightMode" = "Always" }
            
              // 此处是定义通道的代码的其余部分。
        }
    }
}
```

#### 着色器程序（HLSL）：
下面是一个示例结构：HLSLINCLUDE和ENDHLSL之间放置需要共享的代码块。

```
Shader "Examples/ExampleShader"
{
    SubShader
    {

        HLSLINCLUDE
            // 在此编写要共享的 HLSL 代码
        ENDHLSL

        Pass
        {                
              Name "ExampleFirstPassName"
              Tags { "LightMode" = "ExampleLightModeTagValue" }

              // 在此编写设置渲染状态的 ShaderLab 命令

              HLSLPROGRAM
                // 此 HLSL 着色器程序自动包含上面的 HLSLINCLUDE 块的内容
                // 在此编写 HLSL 着色器代码
              ENDHLSL
        }

        Pass
        {                
              Name "ExampleSecondPassName"
              Tags { "LightMode" = "ExampleLightModeTagValue" }

              // 在此编写设置渲染状态的 ShaderLab 命令

              HLSLPROGRAM
                // 此 HLSL 着色器程序自动包含上面的 HLSLINCLUDE 块的内容
                // 在此编写 HLSL 着色器代码
              ENDHLSL
        }

    }
}
```

#### ShaderLab命令：
ShaderLab 命令分为以下类别：
- 用于在 GPU 上设置渲染状态的命令。
- 用于创建具有特定用途的通道。
- 如果使用旧版 “fixed function style” 命令，无需编写 HLSL 也可创建着色器程序。
可以通过 [Category 代码块](https://docs.unity.cn/cn/2022.3/Manual/SL-Other.html)将 ShaderLab 命令组合起来。
但是现在基本上都用[[Unity中的HLSL]]了
###### 用于设置渲染状态的命令：
在 Pass 代码块中使用这些命令可为该 Pass 设置渲染状态，或者在 SubShader 代码块中使用这些命令可为该 SubShader 以及其中的所有 Pass 设置渲染状态。

- [AlphaToMask](https://docs.unity.cn/cn/2022.3/Manual/SL-AlphaToMask.html)：设置 alpha-to-coverage 模式。
- [Blend](https://docs.unity.cn/cn/2022.3/Manual/SL-Blend.html)：启用和配置 alpha 混合。
- [BlendOp](https://docs.unity.cn/cn/2022.3/Manual/SL-BlendOp.html)：设置 Blend 命令使用的操作。
- [ColorMask](https://docs.unity.cn/cn/2022.3/Manual/SL-ColorMask.html)：设置颜色通道写入掩码。
- [Conservative](https://docs.unity.cn/cn/2022.3/Manual/SL-Conservative.html)：启用和禁用保守光栅化。
- [Cull](https://docs.unity.cn/cn/2022.3/Manual/SL-Cull.html)：设置多边形剔除模式。
- [Offset](https://docs.unity.cn/cn/2022.3/Manual/SL-Offset.html)：设置多边形深度偏移。
- [Stencil](https://docs.unity.cn/cn/2022.3/Manual/SL-Stencil.html)：配置模板测试，以及向模板缓冲区写入的内容。
- [ZClip](https://docs.unity.cn/cn/2022.3/Manual/SL-ZClip.html)：设置深度剪辑模式。
- [ZTest](https://docs.unity.cn/cn/2022.3/Manual/SL-ZTest.html)：设置深度测试模式。
- [ZWrite](https://docs.unity.cn/cn/2022.3/Manual/SL-ZWrite.html)：设置深度缓冲区写入模式。

###### 通道命令：
在 SubShader 中使用这些命令可定义具有特定用途的通道。

- [UsePass](https://docs.unity.cn/cn/2022.3/Manual/SL-UsePass.html) 定义一个通道，它从另一个 Shader 对象导入指定的通道的内容。
- [GrabPass](https://docs.unity.cn/cn/2022.3/Manual/SL-GrabPass.html) 创建一个通道，将屏幕内容抓取到纹理中，以便在之后的通道中使用。但是这个功能好像只在内置渲染管线中存在，再URP中这些高级渲染管线中就已经被取消了。

###### 旧版“固定函数样式”命令：
这些命令的文档在页面 [ShaderLab 旧版功能](https://docs.unity.cn/cn/2022.3/Manual/shader-shaderlab-legacy.html)上。

##### Category代码块对命令进行分组：
使用 **Category** 代码块可对设置渲染状态的命令进行分组，这样您可以“继承”该代码块内的分组渲染状态。
例如，您的 Shader 对象可能有多个[子着色器](https://docs.unity.cn/cn/2022.3/Manual/SL-SubShader.html)，每个都需要[混合](https://docs.unity.cn/cn/2022.3/Manual/SL-Blend.html)设置为加法。可以如下所示使用 Category 代码块：

```
Shader "example" {
Category {
    Blend One One
    SubShader {
        // ...
    }
    SubShader {
        // ...
    }
    // ...
}
}
```
Category 代码块对着色器性能没有影响；它本质上与复制粘贴代码相同。

##### ShaderLab命令：AlphaToMask
启用或禁用 GPU 上的 [alpha-to-coverage](https://en.wikipedia.org/wiki/Alpha_to_coverage) 模式。
Alpha-to-coverage 模式可以减少将多样本抗锯齿 (MSAA) 与使用 Alpha 测试的着色器（如植被着色器）一起使用时出现的过度锯齿。为此，它根据片元着色器结果输出中的 Alpha 值按比例修改多样本覆盖率遮罩。
此命令旨在与 MSAA 一起使用。如果在不使用 MSAA 时启用 alpha-to-coverage 模式，结果无法预测；不同的图形 API 和 GPU 对此有不同的处理方式。

所有管线都适用。

|**参数**|**值**|**功能**|
|---|---|---|
|**state**|`On`|启用 alpha-to-coverage 模式。|
||`Off`|禁用 alpha-to-coverage 模式。|
此实例示例代码演示在Pass中的命令。

```
Shader "Examples/CommandExample"
{
    SubShader
    {
         // 此处是定义子着色器的代码的其余部分。

        Pass
        {    
              // 为此通道启用减法混合
              AlphaToMask On
            
              // 此处是定义通道的代码的其余部分。
        }
    }
}
```

此示例代码演示在 SubShader 代码块中使用此命令的语法。

```
Shader "Examples/CommandExample"
{
    SubShader
    {
         // 为此子着色器启用 alpha-to-coverage 模式
         AlphaToMask On

         // 此处是定义子着色器的代码的其余部分。

        Pass
        {    
              // 此处是定义通道的代码的其余部分。
        }
    }
}
```

##### ShaderLab命令：Blend
确定 GPU 如何将片元着色器的输出与渲染目标进行合并。
此命令的功能取决于混合操作，您可以使用 BlendOp 命令进行设置。请注意，虽然所有图形 API 和硬件都支持混合功能，但对某些混合操作的支持较为有限。
启用混合会禁用 GPU 上的一些优化（主要是删除隐藏的表面/Early-Z），这些优化会增加 GPU 帧时间。
基本上所有的透明效果都要用Blend，也是所有的管线都支持。
此命令会更改渲染状态。在 `Pass` 代码块中使用它可为该通道设置渲染状态，或者在 `SubShader` 代码块中使用它可为该子着色器中的所有通道设置渲染状态。

如果启用了混合，则会发生以下情况：

- 如果使用 BlendOp 命令，则混合操作将设置为该值。否则，混合操作默认为 `Add`。
- 如果混合操作是 `Add`、`Sub`、`RevSub`、`Min` 或 `Max`，GPU 会将片元着色器的输出值乘以源系数。
- 如果混合操作是 `Add`、`Sub`、`RevSub`、`Min` 或 `Max`，GPU 会将渲染目标中现有的值乘以目标系数。
- GPU 对结果值执行混合操作。

混合等式为：

```
finalValue = (sourceFactor * sourceValue) operation (destinationFactor * destinationValue)
```

在这个等式中：

- `finalValue` 是 GPU 写入目标缓冲区的值。
- `sourceFactor` 在 Blend 命令中定义。
- `sourceValue` 是片元着色器输出的值。
- `operation` 是混合操作。
- `destinationFactor` 在 Blend 命令中定义。
- `destinationValue` 是目标缓冲区中现有的值。

|**签名**|**示例语法**|**功能**|
|---|---|---|
|`Blend <state>`|`Blend Off`|禁用默认渲染目标的混合。这是默认值。|
|`Blend <render target> <state>`|`Blend 1 Off`|如上，但针对给定的渲染目标。(1)|
|`Blend <source factor> <destination factor>`|`Blend One Zero`|启用默认渲染目标的混合。设置 RGBA 值的混合系数。|
|`Blend <render target> <source factor> <destination factor>`|`Blend 1 One Zero`|如上，但针对给定的渲染目标。(1)|
|`Blend <source factor RGB> <destination factor RGB>, <source factor alpha> <destination factor alpha>`|`Blend One Zero, Zero One`|启用默认渲染目标的混合。为 RGB 和 Alpha 值设置单独的混合系数。(2)|
|`Blend <render target> <source factor RGB> <destination factor RGB>, <source factor alpha> <destination factor alpha>`|`Blend 1 One Zero, Zero One`|如上，但针对给定的渲染目标。(1) (2)|

**注意：**

1. 任何指定渲染目标的签名都需要 OpenGL 4.0+、`GL_ARB_draw_buffers_blend` 或 OpenGL ES 3.2。
2. 单独的 RGB 和 Alpha 混合与[高级 OpenGL 混合操作](https://docs.unity.cn/cn/2022.3/Manual/SL-BlendOp.html)不兼容。

###### 有效参数值

|**参数**|**值**|**功能**|
|---|---|---|
|**render target**|整数，范围 0 到 7|渲染目标索引。|
|**state**|`Off`|禁用混合。|
|**factor**|`One`|此输入的值是 one。该值用于使用源或目标的颜色的值。|
||`Zero`|此输入的值是 zero。该值用于删除源或目标值。|
||`SrcColor`|GPU 将此输入的值乘以源颜色值。|
||`SrcAlpha`|GPU 将此输入的值乘以源 Alpha 值。|
||`SrcAlphaSaturate`|The GPU multiplies the value of this input by the minimum value of `source alpha` and `(1 - destination alpha)`|
||`DstColor`|GPU 将此输入的值乘以帧缓冲区的源颜色值。|
||`DstAlpha`|GPU 将此输入的值乘以帧缓冲区的源 Alpha 值。|
||`OneMinusSrcColor`|GPU 将此输入的值乘以（1 - 源颜色）。|
||`OneMinusSrcAlpha`|GPU 将此输入的值乘以（1 - 源 Alpha）。|
||`OneMinusDstColor`|GPU 将此输入的值乘以（1 - 目标颜色）。|
||`OneMinusDstAlpha`|GPU 将此输入的值乘以（1 - 目标 Alpha）。|

###### 常见混合类型 (Blend Type)

以下是最常见的混合类型的语法：

```
Blend SrcAlpha OneMinusSrcAlpha // 传统透明度
Blend One OneMinusSrcAlpha // 预乘透明度
Blend One One // 加法
Blend OneMinusDstColor One // 软加法
Blend DstColor Zero // 乘法
Blend DstColor SrcColor // 2x 乘法
```

###### 示例

```
Shader "Examples/CommandExample"
{
    SubShader
    {
         // 此处是定义子着色器的代码的其余部分。

        Pass
        {    
              // 为此通道启用常规 Alpha 混合
      Blend SrcAlpha OneMinusSrcAlpha
            
              // 此处是定义通道的代码的其余部分。
        }
    }
}
```

此示例代码演示在 SubShader 代码块中使用此命令的语法。

```
Shader "Examples/CommandExample"
{
    SubShader
    {
         // 为此子着色器启用常规 Alpha 混合
         Blend SrcAlpha OneMinusSrcAlpha

         // 此处是定义子着色器的代码的其余部分。

        Pass
        {    
              // 此处是定义通道的代码的其余部分。
        }
    }
}
```
##### ShaderLab命令：ColorMask
设置颜色通道写入遮罩，以防止 GPU 写入渲染目标中的通道。
默认情况下，GPU 写入所有通道 (RGBA)。对于某些效果，您可能希望不修改某些通道；例如，您可以禁用颜色渲染来渲染无色阴影。另一个常见的用例是完全禁用颜色写入，以便您在一个缓冲区中填充数据而无需写入其他缓冲区；例如，您可能需要在不写入渲染目标的情况下填充模板缓冲区。
所有的渲染管线都可以使用。

此命令会更改渲染状态。在 `Pass` 代码块中使用它可为该通道设置渲染状态，或者在 `SubShader` 代码块中使用它可为该子着色器中的所有通道设置渲染状态。

|**签名**|**示例语法**|**功能**|
|---|---|---|
|`ColorMask <channels>`|`ColorMask RGB`|写入默认渲染目标的给定通道。|
|`ColorMask <channels> <render target>`|`ColorMask RGB 2`|如上，但针对给定的渲染目标。|

|**参数**|**值**|**功能**|
|---|---|---|
|**render target**|整数，0 到 7。|渲染目标索引。|
|**channels**|`0`|启用对 R、G、B 和 A 通道的颜色写入。|
||`R`|启用对红色通道的颜色写入。|
||`G`|启用对绿色通道的颜色写入。|
||`B`|启用对蓝色通道的颜色写入。|
||`A`|启用对 Alpha 通道的颜色写入。|
||`R`、`G`、`B` 和 `A` 的任意组合，无空格。例如：`RB`|启用对给定通道的颜色写入。|
```
Shader "Examples/CommandExample"
{
    SubShader
    {
         // 此处是定义子着色器的代码的其余部分。

        Pass
        {    
              // 对此通道启用仅写入 RGB 通道，这会禁用向 Alpha 通道写入
              ColorMask RGB

              // 此处是定义通道的代码的其余部分
        }
    }
}
```

```
Shader "Examples/CommandExample"
{
    SubShader
    {
         // 对此通道启用仅写入 RGB 通道，这会禁用向 Alpha 通道写入
         ColorMask RGB

         // 此处是定义子着色器的代码的其余部分。

        Pass
        {    
              // 此处是定义通道的代码的其余部分。
        }
    }
}
```


##### ShaderLab命令：Conservative
启用或禁用保守光栅化。

光栅化是一种渲染技术，通过确定哪些像素被三角形覆盖，将矢量数据（三角形投影）转换为像素数据（渲染目标）。通常情况下，GPU 通过对像素内的点进行采样，判断是否被三角形覆盖来确定是否对像素进行光栅化；如果覆盖范围足够，则 GPU 确定该像素被覆盖。保守光栅化是指 GPU 对被三角形部分覆盖的像素进行光栅化，无论覆盖范围如何。这在需要确定性时很有用，例如在执行遮挡剔除、GPU 上的碰撞检测或可见性检测时。

保守光栅化意味着 GPU 在三角形边上生成更多的片元；这会导致更多片元着色器调用，从而导致 GPU 帧时间增加。只有在特定的场景下才会使用这个技术

##### ShaderLab命令：Cull
设置 GPU 应该基于多边形相对于摄像机的方向剔除哪些多边形。
剔除是确定不绘制什么的过程。剔除提高了渲染效率，因为不会浪费 GPU 时间来绘制在最终图像中不可见的内容。
默认情况下，GPU 执行背面剔除；这意味着它不绘制背对观察者的多边形。一般来说，渲染工作量减少得越多越好；因此，只在必要时才应更改此设置。

| **参数**    | **值**   | **功能**                                         |
| --------- | ------- | ---------------------------------------------- |
| **state** | `Back`  | 剔除背对摄像机的多边形。这称为背面剔除。  <br>  <br>这是默认值。         |
|           | `Front` | 剔除面向摄像机的多边形。这称为正面剔除。  <br>  <br>使用它可翻转几何体。     |
|           | `Off`   | 不根据面朝的方向剔除多边形。  <br>  <br>可用于实现特殊效果，如透明对象或双面墙。 |
##### ShaderLab命令：Offset
设置 GPU 上的深度偏差。
深度偏差，也称为深度偏移，是 GPU 上的一个设置，决定了 GPU 绘制几何体的深度。调整深度偏差以强制 GPU 在具有相同深度的其他几何体之上绘制几何体。这可以帮助您避免不需要的视觉效果，例如深度冲突和阴影暗斑。
要为特定几何体设置深度偏差，请使用此命令或 [RenderStateBlock](https://docs.unity.cn/cn/current/ScriptReference/Rendering.RenderStateBlock.html) 。要设置影响所有几何体的全局深度偏差，请使用 [CommandBuffer.SetGlobalDepthBias](https://docs.unity.cn/cn/current/ScriptReference/Rendering.CommandBuffer.SetGlobalDepthBias.html)。除了全局深度偏差之外，GPU 还为特定几何体应用深度偏差。
为了减少阴影暗斑，您可以使用 **light bias** 设置实现类似的视觉效果；但是，这些设置的工作方式不同，并且不会更改 GPU 上的状态。有关更多信息，请参阅[阴影故障排除](https://docs.unity.cn/cn/current/Manual/ShadowPerformance.html)。

命令会更改渲染状态。在 `Pass` 代码块中使用它可为该通道设置渲染状态，或者在 `SubShader` 代码块中使用它可为该子着色器中的所有通道设置渲染状态。

|**签名**|**示例语法**|**功能**|
|---|---|---|
|`Offset <factor>, <units>`|`Offset 1, 1`|根据给定的值，将几何体绘制得更靠近或更远离摄像机。|

| **参数**     | **值**          | **功能**                                                                                                            |
| ---------- | -------------- | ----------------------------------------------------------------------------------------------------------------- |
| **factor** | 浮点数，范围 –1 到 1。 | 缩放最大 Z 斜率，也称为深度斜率，以生成每个多边形的可变深度偏移。  <br>  <br>不平行于近剪裁平面和远剪裁平面的多边形具有 Z 斜率。调整此值以避免此类多边形上出现视觉瑕疵。                     |
| **units**  | 浮点数，范围 –1 到 1。 | 缩放最小可分辨深度缓冲区值，以产生恒定的深度偏移。最小可分辨深度缓冲区值（一个 _unit_）因设备而异。  <br>  <br>负值意味着 GPU 将多边形绘制得更靠近摄像机。正值意味着 GPU 将多边形绘制得更远离摄像机。 |
##### ShaderLab命令：Stencil
配置与 GPU 上的模板缓冲区相关的设置。
模板缓冲区为帧缓冲区中的每个像素存储一个 8 位整数值。为给定像素执行片元着色器之前，GPU 可以将模板缓冲区中的当前值与给定参考值进行比较。这称为模板测试。如果模板测试通过，则 GPU 会执行深度测试。如果模板测试失败，则 GPU 会跳过对该像素的其余处理。这意味着可以使用模板缓冲区作为遮罩来告知 GPU 要绘制的像素以及要丢弃的像素。
通常会将模板缓冲区用于特殊效果，例如门户或镜子。此外，在渲染硬阴影或者[构造型实体几何 (CSG)](https://en.wikipedia.org/wiki/Constructive_solid_geometry?) 时，有时会使用模板缓冲区。
###### 用法：
使用模板测试可以做两样事情：
1、配置模板测试
2、配置GPU对模板缓冲的写入操作
你可以同一个命令里面做这两个事情，但是通常的做法并不是这样。通常的做法是：用一个shader绘制出一片区域，然后让别的shader都无法再在其中绘制。如何做到？第一个shader中让物体永远通过模板测试并且写入模板缓冲区，然后然别的shader进行模板测试并且关闭其对模板缓冲区的写入。
模板测试方程为：

```
(ref & readMask) comparisonFunction (stencilBufferValue & readMask)
```

|**签名**|**示例语法**|**功能**|
|---|---|---|
|`Stencil`  <br>`{`  <br>    `Ref <ref>`  <br>    `ReadMask <readMask>`  <br>    `WriteMask <writeMask>`  <br>    `Comp <comparisonOperation>`  <br>    `Pass <passOperation>`  <br>    `Fail <failOperation>`  <br>    `ZFail <zFailOperation>`  <br>    `CompBack <comparisonOperationBack>`  <br>    `PassBack <passOperationBack>`  <br>    `FailBack <failOperationBack>`  <br>    `ZFailBack <zFailOperationBack>`  <br>    `CompFront <comparisonOperationFront>`  <br>    `PassFront <passOperationFront>`  <br>    `FailFront <failOperationFront>`  <br>    `ZFailFront <zFailOperationFront>`  <br>`}`  <br>  <br>请注意，所有参数都是可选的。|`Stencil`  <br>`{`  <br>    `Ref 2`  <br>    `Comp equal`  <br>    `Pass keep`  <br>    `ZFail decrWrap`  <br>`}`|根据给定参数配置模板缓冲区。|

###### 有效参数值

|**参数**|**值**|**功能**|
|---|---|---|
|**ref**|整数。范围为 0 到 255。默认值为 0。|参考值。  <br>  <br>GPU 使用在 compareOperation 中定义的操作将模板缓冲区的当前内容与此值进行比较。  <br>  <br>此值使用 readMask 或 writeMask 进行遮罩，具体取决于进行的是读取操作还是写入操作。  <br>  <br>如果 Pass、Fail 或 ZFail 的值为 Replace，则 GPU 也可以将此值写入模板缓冲区。|
|**readMask**|整数。范围为 0 到 255。默认值为 255。|GPU 在执行模板测试时使用此值作为遮罩。  <br>  <br>有关模板测试方程，请参阅上文。|
|**writeMask**|整数。范围为 0 到 255。默认值为 255。|GPU 在写入模板缓冲区时使用此值作为遮罩。  <br>  <br>请注意，与其他遮罩一样，它指定操作中包含的位。例如，值为 0 表示写入操作中不包含任何位，而不是模板缓冲区接收值 0。|
|**comparisonOperation**|比较操作。请参阅[比较操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#comparison-operation-values)以了解有效值。默认值为 Always。|GPU 为所有像素的模板测试执行的操作。  <br>  <br>这会定义适用于所有像素的操作，而与朝向无关。如果定义了此值以及 comparationOperationBack 和 comparationOperationFront，则此值会覆盖它们。|
|**passOperation**|模板操作。请参阅[模板操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#stencil-operation-values)以了解有效值。默认值为 Keep。|当像素通过模板测试和深度测试时，GPU 对模板缓冲区执行的操作。  <br>  <br>这会定义适用于所有像素的操作，而与朝向无关。如果定义了此值以及 passOperationBack 和 passOperationFront，则此值会覆盖它们。|
|**failOperation**|模板操作。请参阅[模板操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#stencil-operation-values)以了解有效值。默认值为 Keep。|当像素未能通过模板测试时，GPU 对模板缓冲区执行的操作。  <br>  <br>这会定义适用于所有像素的操作，而与朝向无关。如果定义了此值以及 failOperationBack 和 failOperationFront，则此值会覆盖它们。|
|**zFailOperation**|模板操作。请参阅[模板操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#stencil-operation-values)以了解有效值。默认值为 Keep。|当像素通过模板测试，但是未能通过深度测试时，GPU 对模板缓冲区执行的操作。  <br>  <br>这会定义适用于所有像素的操作，而与朝向无关。如果定义了此值以及 zFailOperation 和 zFailOperation，则此值会覆盖它们。|
|**comparisonOperationBack**|比较操作。请参阅[比较操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#comparison-operation-values)以了解有效值。默认值为 Always。|GPU 为模板测试执行的操作。  <br>  <br>这会定义仅适用于背面像素的操作。如果定义了 comparisonOperation，则该值会覆盖此值。|
|**passOperationBack**|模板操作。请参阅[模板操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#stencil-operation-values)以了解有效值。默认值为 Keep。|当像素通过模板测试和深度测试时，GPU 对模板缓冲区执行的操作。  <br>  <br>这会定义仅适用于背面像素的操作。如果定义了 passOperation，则该值会覆盖此值。|
|**failOperationBack**|模板操作。请参阅[模板操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#stencil-operation-values)以了解有效值。默认值为 Keep。|当像素未能通过模板测试时，GPU 对模板缓冲区执行的操作。  <br>  <br>这会定义仅适用于背面像素的操作。如果定义了 failOperation，则该值会覆盖此值。|
|**zFailOperationBack**|模板操作。请参阅[模板操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#stencil-operation-values)以了解有效值。默认值为 Keep。|当像素通过模板测试，但未能通过深度测试时，GPU 对模板缓冲区执行的操作。  <br>  <br>这会定义仅适用于背面像素的操作。如果定义了 zFailOperation，则该值会覆盖此值。|
|**comparisonOperationFront**|比较操作。请参阅[比较操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#comparison-operation-values)以了解有效值。默认值为 Always。|GPU 为模板测试执行的操作。  <br>  <br>这会定义仅适用于正面像素的操作。如果定义了 comparisonOperation，则该值会覆盖此值。|
|**passOperationFront**|模板操作。请参阅[模板操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#stencil-operation-values)以了解有效值。默认值为 Keep。|当像素通过模板测试和深度测试时，GPU 对模板缓冲区执行的操作。  <br>  <br>这会定义仅适用于正面像素的操作。如果定义了 passOperation，则该值会覆盖此值。|
|**failOperationFront**|模板操作。请参阅[模板操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#stencil-operation-values)以了解有效值。默认值为 Keep。|当像素未能通过模板测试时，GPU 对模板缓冲区执行的操作。  <br>  <br>这会定义仅适用于正面像素的操作。如果定义了 failOperation，则该值会覆盖此值。|
|**zFailOperationFront**|模板操作。请参阅[模板操作值](https://docs.unity.cn/cn/current/Manual/SL-Stencil.html#stencil-operation-values)以了解有效值。默认值为 Keep。|当像素通过模板测试，但未能通过深度测试时，GPU 对模板缓冲区执行的操作。  <br>  <br>这会定义仅适用于正面像素的操作。如果定义了 zFailOperation，则该值会覆盖此值。|

###### 比较操作值

在 C# 中，这些值通过 [Rendering.CompareFunction](https://docs.unity.cn/cn/current/ScriptReference/Rendering.CompareFunction.html) 枚举进行表示。

|**值**|**Rendering.CompareFunction 枚举中的对应整数值**|**功能**|
|---|---|---|
|`Never`|1|从不渲染像素。|
|`Less`|2|在参考值小于模板缓冲区中的当前值时渲染像素。|
|`Equal`|3|在参考值等于模板缓冲区中的当前值时渲染像素。|
|`LEqual`|4|在参考值小于或等于模板缓冲区中的当前值时渲染像素。|
|`Greater`|5|在参考值大于模板缓冲区中的当前值时渲染像素。|
|`NotEqual`|6|在参考值与模板缓冲区中的当前值不同时渲染像素。|
|`GEqual`|7|在参考值大于或等于模板缓冲区中的当前值时渲染像素。|
|`Always`|8|始终渲染像素。|

###### 模板操作值

In C#, these values are represented by the [Rendering.Rendering.StencilOp](https://docs.unity.cn/cn/current/ScriptReference/Rendering.StencilOp.html) enum.

|**值**|**Rendering.StencilOp 枚举中的对应整数值**|**功能**|
|---|---|---|
|`Keep`|0|保持模板缓冲区的当前内容。|
|`Zero`|1|将 0 写入模板缓冲区。|
|`Replace`|2|将参考值写入缓冲区。|
|`IncrSat`|3|递增缓冲区中的当前值。如果该值已经是 255，则保持为 255。|
|`DecrSat`|4|递减缓冲区中的当前值。如果该值已经是 0，则保持为 0。|
|`Invert`|5|将缓冲区中当前值的所有位求反。|
|`IncrWrap`|6|递增缓冲区中的当前值。如果该值已经是 255，则变为 0。|
|`DecrWrap`|7|递减缓冲区中的当前值。如果该值已经是 0，则变为 255。|

###### 示例

```
Shader "Examples/CommandExample"
{
    SubShader
    {
         // 此处是定义子着色器的代码的其余部分。

        Pass
        {    
             // 此通道中的所有像素都会通过模板测试并将值 2 写入模板缓冲区
             // 如果要防止后续着色器绘制到渲染目标的此区域或将它们限制为仅渲染到此区域，则通常会执行此操作
             Stencil
             {
                 Ref 2
                 Comp Always
                 Pass Replace
             }            

             // 此处是定义通道的代码的其余部分。
        }
    }
}
```

此示例代码演示在 SubShader 代码块中使用此命令的语法。

```
Shader "Examples/CommandExample"
{
    SubShader
    {
             // 仅当模板缓冲区的当前值小于 2 时，此子着色器中的所有像素才通过模板测试
             // 如果希望仅绘制到渲染目标中未"遮罩"的区域，则通常会执行此操作
             Stencil
             {
                 Ref 2
                 Comp Less
             }  

         // 此处是定义子着色器的代码的其余部分。

        Pass
        {    
              // 此处是定义通道的代码的其余部分。
        }
    }
}
```
##### ShaderLab命令：UsePass
UsePass 命令插入来自另一个 Shader 对象的指定通道。可以使用此命令来减少着色器源文件中的代码重复。
有关在 ShaderLab 代码中向通道添加名称的信息，请参阅 [ShaderLab：向通道添加名称](https://docs.unity.cn/cn/current/Manual/SL-Name.html)。

| **签名**                                                | **功能**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `UsePass "Shader object name/PASS NAME IN UPPERCASE"` | Inserts the named Pass from the named Shader object.  <br>  <br>If the named Shader object contains more than one SubShader, Unity iterates over the SubShaders until it finds the first supported SubShader that contains a Pass with the given name. For information on how Unity determines whether a SubShader is supported, see [Shader objects introduction](https://docs.unity.cn/cn/current/Manual/shader-objects.html).  <br>  <br>If the SubShader contains more than one Pass with the same name, Unity returns the last Pass it finds.  <br>  <br>If Unity does not find a matching Pass, it shows the [error shader](https://docs.unity.cn/cn/current/Manual/shader-error.html). |
此示例代码创建一个名为 NamedPass 的 Shader 对象，其中包含名为 ExampleNamedPass 的通道。

```
Shader "Examples/ContainsNamedPass"
{
    SubShader
    {
        Pass
        {    
              Name "ExampleNamedPass"
            
              // 此处是通道内容的其余部分。
        }
    }
}
```

此示例代码创建一个名为 UseNamedPass 的 Shader 对象，该对象使用上述示例代码中的指定通道。

```
Shader "Examples/UsesNamedPass"
{
    SubShader
    {
        UsePass "Examples/ContainsNamedPass/EXAMPLENAMEDPASS"
    }
}
```

##### ShaderLab命令：ZClip
##### ShaderLab命令：ZTest
##### ShaderLab命令：ZWrite
