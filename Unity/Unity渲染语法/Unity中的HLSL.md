
本手册的这一部分介绍有关以 Unity 特定方式使用 HLSL 的信息。有关编写 HLSL 的常规信息，请参阅 [Microsoft 的 HLSL 文档](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl)。

将 HLSL 代码放在 ShaderLab 代码中的代码块中。着色器程序通常如下所示：

```
  Pass {
        // ... 常规通道状态设置 ...
      
        HLSLPROGRAM
        // 此代码片段的编译指令，例如：
        #pragma vertex vert
        #pragma fragment frag
      
        // 着色器程序本身
      
        ENDHLSL

        // ... 通道的剩余部分 ...
    }
```

有关着色器代码块的更多信息，请参阅 [ShaderLab：添加着色器程序](https://docs.unity.cn/cn/current/Manual/shader-shaderlab-code-blocks.html)。

## HLSL 语法

HLSL 语言有两种语法：旧版的 DirectX 9 样式语法以及更现代的 DirectX 10+ 样式语法。不同之处主要在于纹理采样函数的工作方式：

- 旧版语法使用 sampler2D、tex2D() 和类似函数。此语法适用于所有平台。
- DX10+ 语法使用 Texture2D、SamplerState 和 .Sample() 函数。由于纹理和采样器在 OpenGL 中不是不同对象，因此该语法的某些形式在 OpenGL 平台上无效。

Unity 提供了包含预处理器宏的着色器库来帮助您管理这些差异。有关更多信息，请参阅[内置着色器宏](https://docs.unity.cn/cn/current/Manual/SL-BuiltinMacros.html)。

## HLSL中的预处理器指令
在hlsl内部，着色器编译有多个阶段。
第一阶段是预处理，称为预处理器的程序为编译准备代码。预处理器指令是针对预处理器的指令。本手册的这一部分包含了Unity特定的使用HLSL预处理器指令的方法，以及Unity独有的HLSL预处理器指令的信息。它不包含关于HLSL支持的所有预处理器指令的详尽文档，也不包含关于在HLSL中使用预处理器指令的一般信息。有关该信息，请参阅HLSL文档： [Preprocessor directives (HLSL)](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-appendix-preprocessor).。

|**页面**|**描述**|
|---|---|
|[include and include_with_pragmas directives in HLSL](https://docs.unity.cn/cn/current/Manual/shader-include-directives.html)|Working with `#include` and the Unity-specific `#include_with_pragmas` directives in HLSL in Unity.|
|[pragma directives in HLSL](https://docs.unity.cn/cn/current/Manual/SL-PragmaDirectives.html)|Working with `#pragma` directives in HLSL in Unity.|
|[Targeting shader models and GPU features in HLSL](https://docs.unity.cn/cn/current/Manual/SL-ShaderCompileTargets.html)|Using `#pragma` directives to indicate that your shader requires certain GPU features.|
|[Targeting graphics APIs and platforms in HLSL](https://docs.unity.cn/cn/current/Manual/SL-ShaderCompilationAPIs.html)|Using `#pragma` directives to target specific graphics API and platforms.|
|[Declaring and using shader keywords in HLSL](https://docs.unity.cn/cn/current/Manual/SL-MultipleProgramVariants.html)|Using `#pragma` directives to declare shader keywords and `#if` directives to indicate that code depends on the state of shader keywords.|
### Include and include_with_pragmas directives in HLSL

在HLSL中，#include指令是一种预处理器指令。它们指示编译器将一个HLSL文件的内容包含在另一个HLSL文件中。它们包含的文件称为包含文件。在Unity中，常规的#include指令与标准HLSL中的工作方式相同。有关常规#include指令的更多信息，请参阅HLSL文档： [include Directive](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-appendix-pre-include)。
Unity还提供了一个额外的，Unity特有的#include_with_pragma。#包括_与_pragma指令的工作原理与普通的#include指令相同，但它也允许你在include文件中使用#pragma指令。这意味着#include_with_pragma指令允许你在多个文件之间共享#pragma指令。
#### 使用#include_with_pragmas指令
注意：使用#include_with_pragmas指令，你必须启用 [Caching Shader Preprocessor](https://docs.unity.cn/cn/current/Manual/shader-compilation.html#preprocessor)。
这个例子演示了如何使用unity特有的#include_with_pragmas指令，以实现一个共同的工作流程改进：切换多个着色器调试开关的能力，而无需每次编辑每个着色器源文件。
#### 为什么这么干？
#include 这一个操作符能包含文件，但是会丢失的#pragma的信息。但是再Unity中，每一个着色器文件中的#pragma文件都很重要，所以Unity提出了#include_with_pragmas这一个指令。和普通的#include一样，只不过这个指令会让被包含的文件里面的#pragma也能工作。
一个很经典的做法是集中管理调试设置：
1. 你创建一个专门的头文件，比如叫 `DebugSymbols.hlsl`，里面只写一行代码：

#pragma enable_d3d11_debug_symbols

2. 在你的10个Shader文件中，都加入这一行：
#include_with_pragmas "DebugSymbols.hlsl"

3. **现在，你想开启或关闭所有10个Shader的调试功能，只需要修改一个文件（`DebugSymbols.hlsl`）即可。** 比如把 `DebugSymbols.hlsl` 里的那行代码注释掉：
 #pragma enable_d3d11_debug_symbols
 
下次编译时，所有10个Shader的调试功能就都关闭了。

### HLSL中的#pragma指令
在HLSL中#pragma是预处理指令。它告诉着色器编译器一下预处理器指令所不包括的附加信息。
理论上你可以放在任何位置，因为它们都在最开始的预处理阶段，编译器会把它们先拉出来，但是通常我们都放在前面。

```
#pragma target 3.0
#pragma exclude_renderers vulkan
#pragma vertex vert
#pragma fragment frag

// The rest of your HLSL code goes here
```

#### 有一些#pragma指令上面的限制：

限制一：在条件编译（#if）中的使用限制
你不能用任意的变量来做条件#pragma，只可以使用：
1、自己定义的宏
2、Unity明确列出的几个平台的宏 `SHADER_API_MOBILE`, `SHADER_API_DESKTOP`, `UNITY_NO_RGBM`, `UNITY_USE_NATIVE_HDR`, `UNITY_FRAMEBUFFER_FETCH_AVAILABLE`, `UNITY_NO_CUBEMAP_ARRAY`， `UNITY_VERSION`。
3、两者结合
原因：编译分阶段，早期Unity只知道你自己定义的宏，还有UNITY的几个平台宏。如果你使用了别的UNITY是无法进行条件判断的。所以UNITY把这几个都禁掉了。
`
`// ✅ 允许：使用自定义宏做条件
#define USE_FANCY_EFFECT 1
#if USE_FANCY_EFFECT
    #pragma multi_compile _ FANCY_ON
#endif

// ✅ 允许：使用平台宏做条件
#if SHADER_API_MOBILE
    #pragma multi_compile _ MOBILE_SHADOWS
#else
    #pragma multi_compile _ DESKTOP_SHADOWS
#endif

// ❌ 不允许：使用未列出的内置宏（如SHADER_TARGET_3_0）做条件
#if SHADER_TARGET_3_0
    #pragma target 3.0 // 编译器可能会忽略此条#pragma
#endif`

限制二：#pragma指令和#include指令的隔离规则
**Unity 将 `#pragma` 分为两种，并且规定它们必须在不同的文件中生效。**

|指令类型|生效场所|被谁处理|
|---|---|---|
|**Unity 特有的 `#pragma`**  <br>(如 `multi_compile`, `vertex`, `fragment`, `target`)|**.shader 文件** 或 **`#include_with_pragmas` 引入的文件**|**Unity 编辑器/编译器**|
|**标准 HLSL 的 `#pragma`**  <br>(如 `pack_matrix`, `message`, `warning`)|**`#include` 引入的文件**|**底层着色器编译器 (如DXC, D3DCompile)**|

**为什么要有这种奇怪的隔离？**
想象一下编译流程：

1. **Unity 预处理**：首先，Unity 编辑器会读取你的 `.shader` 文件。它需要找到所有的 `#pragma vertex`、`#pragma multi_compile` 等指令，来弄清楚要为这个着色器编译几个变体、每个变体用什么函数。
    
2. **生成变体并编译**：然后，Unity 为每个变体生成一份完整的最终代码，并交给底层的着色器编译器（比如微软的 `D3DCompile` 或新的 `DXC`）。
    
3. **底层编译**：底层编译器接收最终代码，进行编译。
     
如果 Unity 在 `#include` 进来的文件中看到了 `#pragma multi_compile`，它可能会感到困惑，因为它不确定是否应该在预处理阶段处理这个指令。  
同样，如果底层编译器在 `.shader` 文件里看到了不认识的 `#pragma pack_matrix`，它也会忽略它。

为了解决这个混乱，Unity 制定了清晰的边界：

- **`.shader` 文件是 Unity 的地盘**：在这里，它只关心和处理自己特有的 `#pragma`，会忽略所有标准 `#pragma`。
    
- **`#include` 引入的文件是底层编译器的地盘**：Unity 在预处理阶段会原封不动地把这些文件的内容复制过去。它自己不会处理里面的任何 `#pragma`。所有这些代码（包括里面的标准 `#pragma`）都会最终交给底层编译器去处理。
    
- **`#include_with_pragmas` 是特例**：这个指令告诉 Unity：“这个文件很重要，麻烦你亲自进来处理一下里面的 **Unity 特有 `#pragma`** 指令”。这样，你就可以把 Unity 的 `#pragma` 指令组织到独立的头文件里了。

| 特性         | Unity 特有 `#pragma`                                                                                                                                                                                      | 标准 HLSL `#pragma`                                |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| **指令类型**   | `multi_compile`, `shader_feature`, `vertex`, `fragment`, `target` 等[](http://www.likecs.com/show-308100255.html)[](https://blog.csdn.net/fanstasic/article/details/112183205)                           | `pack_matrix`, `message`, `warning`, `disable` 等 |
| **处理阶段**   | **Unity 预处理阶段**                                                                                                                                                                                         | **底层着色器编译阶段**                                    |
| **处理者**    | **Unity 编辑器/编译器**                                                                                                                                                                                       | **底层编译器 (如DXC, D3DCompile)**                     |
| **可用场所**   | **.shader 文件** 或 **`#include_with_pragmas` 引入的文件**                                                                                                                                                      | **`#include` 引入的文件**                             |
| **被忽略的场所** | 在 `#include` 引入的文件中                                                                                                                                                                                     | 在 `.shader` 文件 或 `#include_with_pragmas` 引入的文件中  |
| **主要作用**   | 控制**着色器变体生成**、指定着色器函数、设置渲染平台和目标[](http://www.likecs.com/show-308100255.html)[](https://blog.csdn.net/fanstasic/article/details/112183205)[](https://docs.unity.cn/cn/560/Manual/SL-ShaderPrograms.html) | 控制**底层编译器行为**、矩阵打包、发出编译警告/错误等                    |

#### 支持的pragma指令列表

Unity支持所有作为标准hlsl所有的#pragma指令，只要这些指令位于常规的include文件中。有关这些指令的更多信息，请参阅HLSL文档：[pragma Directive](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-appendix-pre-pragma).。

##### 着色器阶段：

| **语句**                    | **功能**                                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `#pragma vertex <name>`   | Compile the function with the given name as the vertex shader. Replace <name> with the function name. This directive is required in regular graphics shaders.                                                                                                                                                                                                                                 |
| `#pragma fragment <name>` | Compile the function with the given name as the fragment shader. Replace <name> with the function name. This directive is required in regular graphics shaders.                                                                                                                                                                                                                               |
| `#pragma geometry <name>` | 几何着色器：Compile the function with the given name as the geometry shader. Replace <name> with the function name. This option automatically turns on `#pragma require geometry`; for more information, see [Targeting shader models and GPU features in HLSL](https://docs.unity.cn/cn/2022.3/Manual/SL-ShaderCompileTargets.html).  <br>  <br>**Note**: Metal does not support geometry shaders. |
| `#pragma hull <name>`     | Compile the function with the given name as the DirectX 11 hull shader. Replace <name> with the function name. This automatically adds `#pragma require tessellation`; for more information, see [Targeting shader models and GPU features in HLSL](https://docs.unity.cn/cn/2022.3/Manual/SL-ShaderCompileTargets.html).                                                                     |
| `#pragma domain <name>`   | Compile the function with the given name as the DirectX 11 domain shader. Replace <name> with the function name. This option automatically turns on `#pragma require tessellation`; for more information, see [Targeting shader models and GPU features in HLSL](https://docs.unity.cn/cn/2022.3/Manual/SL-ShaderCompileTargets.html).                                                        |
##### 着色器变体和关键词

| **Directive**                              | **描述**                                                                                                                                                                                                                                                                                                                                                  |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `#pragma multi_compile <keywords>`         | Declares a collection of keywords. The compiler includes all of the keywords in the build.  <br>You can use suffixes such as `_local` to set additional options.  <br>For more information and a list of supported suffixes, see [Declaring and using shader keywords in HLSL](https://docs.unity.cn/cn/2022.3/Manual/SL-MultipleProgramVariants.html). |
| `#pragma shader_feature <keywords>`        | Declares a collection of keywords. The compiler excludes unused keywords from the build.  <br>You can use suffixes such as `_local` to set additional options.  <br>For more information and a list of supported suffixes, see [Declaring and using shader keywords in HLSL](https://docs.unity.cn/cn/2022.3/Manual/SL-MultipleProgramVariants.html).   |
| `#pragma hardware_tier_variants <values>`  | 不支持URP等高级渲染管线。Built-in Render Pipeline only: Add keywords for **graphics tiers** when compiling for a given graphics API. For more information, see [Graphics tiers](https://docs.unity.cn/cn/2022.3/Manual/graphics-tiers.html).                                                                                                                       |
| `#pragma skip_variants <list of keywords>` | Strip specified keywords.                                                                                                                                                                                                                                                                                                                               |
##### GPU支持

Use these directives to tell the compiler that your shader requires specific GPU features.

|**语句**|**功能**|
|---|---|
|`#pragma target <value>`|The minimum shader model that this shader program is compatible with. Replace <value> with a valid value. For a list of valid values, see [Shader compilation: Targeting shader models and GPU features in HLSL](https://docs.unity.cn/cn/2022.3/Manual/SL-ShaderCompileTargets.html).|
|`#pragma require <value>`|The minimum GPU features that this shader is compatible with. Replace <value> with a valid value, or multiple valid values separated by a space. For a list of valid values, see [Shader compilation: Targeting shader models and GPU features in HLSL](https://docs.unity.cn/cn/2022.3/Manual/SL-ShaderCompileTargets.html).|
##### 图形 API

Use these directives to tell Unity to include or exclude code for a given graphics API.

|**语句**|**功能**|
|---|---|
|`#pragma only_renderers <value>`|Compile this shader program only for given graphics APIs. Replace <values> with a space-delimited list of valid values. For more information and a list of valid values, see [Targeting graphics APIs and platforms in HLSL](https://docs.unity.cn/cn/2022.3/Manual/SL-ShaderCompilationAPIs.html).|
|`#pragma exclude_renderers <value>`|Do not compile this shader program for given graphics APIs. Replace <value> with a space-delimited list of valid values. For more information and a list of valid values, see [Targeting graphics APIs and platforms in HLSL](https://docs.unity.cn/cn/2022.3/Manual/SL-ShaderCompilationAPIs.html).|

##### 其他 pragma 指令

|**语句**|**功能**|
|---|---|
|`#pragma instancing_options <options>`|Enable GPU instancing in this shader, with given options. For more information, see [GPU instancing](https://docs.unity.cn/cn/2022.3/Manual/GPUInstancing.html)|
|`#pragma once`|Put this directive in a file to ensure that the compiler includes the file only once in a shader program.  <br>  <br>**Note:** Unity only supports this directive when the [Caching Shader Preprocessor](https://docs.unity.cn/cn/2022.3/Manual/shader-compilation.html#preprocessor) is enabled.|
|`#pragma enable_d3d11_debug_symbols`|Generates shader debug symbols for supported graphics APIs, and disables optimizations for all graphics APIs. Use this for debugging shader code in an external tool.  <br>  <br>Unity generates debug symbols for Vulkan, DirectX 11 and 12, and supported console platforms.  <br>  <br>**Warning:** Using this results in an increased file size and reduced shader performance. When you have finished debugging your shaders and you are ready to make a final build of your application, remove this line from your shader source code and recompile the shaders.|
|`#pragma skip_optimizations <value>`|Forces optimizations off for given graphics APIs. Replace <values> with a space-delimited list of valid values. For a list of valid values, see [Targeting graphics APIs and platforms in HLSL](https://docs.unity.cn/cn/2022.3/Manual/SL-ShaderCompilationAPIs.html)|
|`#pragma hlslcc_bytecode_disassembly`|将反汇编的 HLSLcc 字节码嵌入到转换的着色器中。|
|`#pragma disable_fastmath`|启用涉及 NaN 处理的精确 IEEE 754 规则。当前这仅影响 Metal 平台。|
|`#pragma editor_sync_compilation`|强制进行同步编译。这仅影响 Unity 编辑器。有关更多信息，请参阅[异步着色器编译](https://docs.unity.cn/cn/2022.3/Manual/AsynchronousShaderCompilation.html)。|
|`#pragma enable_cbuffer`|使用 HLSLSupport 的 `CBUFFER_START(name)` 和 `CBUFFER_END` 宏时，即使当前平台不支持常量缓冲区，也要发出 `cbuffer(name)`。|
### HLSL中的Shader model和GPU features
这一个功能是对于GPU进行的筛选，像是一个功能套餐。默认的是#pragma target 2.5，“着色器模型”（Shader Model，如 3.0, 4.0, 5.0）理解为一个**功能套餐**。指定一个更高的目标，就相当于要求了一整套更高级的功能。
一共有两种筛选方案，一种是#pragma target，另一种是#pragma require。这两种分别对应的就是Shader model和GPU features。
使用#pragm target这个比较简单一些，就是从数字小到大排序，依次要求变高。这个和LOD正好是反着来的。默认值是2.5。
还有另一种方式是#pragma require这种指令，比如说我想要使用几何着色器或者细分着色器，就需要写
`#pragma require geomtry`
如果你没有写好这个require而直接使用了`#pragma geometry`的话，Unity也会给你自动补上require，但是同时untiy会给你发一个警告，为了以后debug能更加容易一些，最好都加上。
类似的还有
`// 你想要使用几何着色器
#pragma geometry geom // 声明几何着色器函数
// 为了避免Unity的警告，并且确保代码清晰，你自己最好加上：
#pragma require geometry
// 或者直接指定一个包含此功能的套餐：
#pragma target 4.0`

总的来说：这些#pragma基本上都是在Pass代码块之外，而在Subshader之内，和LOD功能上都是为了对显卡进行挑选。如果使用#pragma target xxx的话就是升序排列，而LOD是降序的。不同的是#pragma对于控制的更加精确，需要你诚实得对照好这个Subshader里面的Pass到底需要显卡做到哪些功能，而LOD只是在顺序上面控制一些先编译哪一个。
##### 指定Shader model或GPU features
怎么指定Shader model？用#pragma target
怎么指定GPU features？用#pragma require ……
其实这两个在做的都是同一个事情，就是上面说对于GPU的筛选。
值得注意的一点是，在大项目里面，对于不同的显卡设计不同的着色器。想象一个场景：你有一个高级的材质，它有一个 `_ENABLE_FANCY_EFFECT` 关键字。
- **当 `_ENABLE_FANCY_EFFECT` 关闭时**：它只使用一些基础指令，可以在任何老旧的手机上运行。
    
- **当 `_ENABLE_FANCY_EFFECT` 开启时**：它使用了一些非常现代的高级功能（比如 `mrt8` - 8个渲染目标），这需要新的硬件才能支持。
按照以前的写法，你不得不将开启特效的变体也设为低目标（`target 2.5`），这会导致编译错误；或者将整个 `SubShader` 设为高目标（`target 4.0`），这会导致低端设备根本无法运行这个材质，即使它们根本不会开启那个高级特效。
###### 1. 用于 `#pragma require` (按功能“单点”)

**a) 无条件要求（对所有变体生效）**
#pragma require integers
// 这意味着：编译这个SubShader的【所有】变体时，都要求硬件支持“整数”功能。

**b) 有条件要求（仅对特定关键字的变体生效）**
#pragma require integers mrt8 : EXAMPLE_KEYWORD OTHER_EXAMPLE_KEYWORD
// 这意味着：只有当编译【启用了 EXAMPLE_KEYWORD 或 OTHER_EXAMPLE_KEYWORD】的变体时，
// 才要求硬件支持“整数”和“8个渲染目标（mrt8）”功能。
// 对于没有启用这些关键字的变体，则没有这个额外要求。

**你可以混合使用：**
#pragma require integers // 所有变体都需要整数功能
#pragma require mrt8 : EXAMPLE_KEYWORD // 只有EXAMPLE_KEYWORD变体需要mrt8功能

###### 2. 用于 `#pragma target` (按“套餐”要求)

**a) 无条件要求（对所有变体生效）**
#pragma target 3.0
// 这意味着：编译这个SubShader的【所有】变体时，都要求硬件至少支持Shader Model 3.0的功能集。

**b) 有条件要求（仅对特定关键字的变体生效）**
#pragma target 4.0 EXAMPLE_KEYWORD OTHER_EXAMPLE_KEYWORD
// 注意：这里没有冒号！
// 这意味着：只有当编译【启用了 EXAMPLE_KEYWORD 或 OTHER_EXAMPLE_KEYWORD】的变体时，
// 才要求硬件支持Shader Model 4.0。
// 对于没有启用这些关键字的变体，将使用这个SubShader默认的target（比如2.5）。

下面就是一个例子：
Shader "Examples/VariantSpecificTarget"
{
    SubShader
    {
        // 1. 这是SubShader的默认（后备）硬件要求。
        // 所有不涉及_SUPER_EFFECT关键字的变体，都只需要SM 2.5。
        #pragma target 2.5

        // 2. 明确声明：只有当编译启用_SUPER_EFFECT的变体时，才需要SM 4.0的功能（包含geometry）。
        #pragma target 4.0 _SUPER_EFFECT

        // 3. 定义变体集合
        #pragma multi_compile __ _SUPER_EFFECT

        Pass
        {
            // 对于所有变体都通用的阶段
            #pragma vertex vert
            #pragma fragment frag

            // 4. 条件性地声明几何着色器阶段。
            // 这个#pragma本身也会被条件编译块包裹，确保只有在_SUPER_EFFECT开启时才会被定义。
            #ifdef _SUPER_EFFECT
                #pragma geometry geom
            #endif

            ... // CGPROGRAM etc.

            // 着色器代码中，也会用 #ifdef _SUPER_EFFECT 来包裹使用几何着色器的部分
            ENDCG
        }
    }
}

当然在游戏中还是要用C#脚本来控制：
public Material myMaterial;

void EnableSuperEffect()
{
    myMaterial.EnableKeyword("_SUPER_EFFECT");
}

void DisableSuperEffect()
{
    myMaterial.DisableKeyword("_SUPER_EFFECT");
}

总的来说：
1. **编译时（Build Time/Import Time）：**
    
    - Unity 看到 `#pragma multi_compile __ _SUPER_EFFECT`，知道需要生成 2 个变体。
        
    - 它为第一个变体（`__`）使用 `target 2.5` 进行编译，并且因为 `_SUPER_EFFECT` 未定义，所有 `#ifdef _SUPER_EFFECT` 块内的代码（几何着色器和昂贵特效）都**被预处理器移除**了。生成的代码非常轻量。
        
    - 它为第二个变体（`_SUPER_EFFECT`）使用 `target 4.0` 进行编译，并且包含所有高级代码。
        
2. **运行时（Run Time）：**
    
    - 当游戏运行时，Unity 会检查当前设备的硬件能力。
        
    - 当你调用 `material.EnableKeyword("_SUPER_EFFECT")` 时，Unity 会：
        
        - **首先**，检查设备是否支持 `target 4.0`（即是否支持几何着色器等）。
            
        - **如果支持**，它就使用那个包含了所有高级代码的、为 `_SUPER_EFFECT` 关键字编译的变体。
            
        - **如果不支持**，这个操作会失败（或者回退到默认变体），从而避免了在低端设备上运行不支持的代码而导致崩溃或错误。
##### Shader model（#pragma target）的值的列表
|**值**|**描述**|**支持**|**Equivalent `#pragma require` values**|
|---|---|---|---|
|`2.0`|Equivalent to [DirectX shader model 2.0](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-sm2).  <br>  <br>Limited amount of arithmetic and texture instructions; 8 interpolators; no vertex texture sampling; no derivatives in fragment shaders; no explicit LOD texture sampling.|Works on all platforms supported by Unity.|无|
|`2.5`|Almost the same as 3.0, but with only 8 interpolators, and no explicit LOD texture sampling.|DirectX 11 feature level 9+  <br>OpenGL 3.2+  <br>OpenGL ES 2.0  <br>Vulkan  <br>Metal|`derivatives`|
|`3.0`|Equivalent to [DirectX shader model 3.0](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-sm3).  <br>  <br>.|DirectX 11 feature level 10 +  <br>OpenGL 3.2+  <br>OpenGL ES 3.0+  <br>Vulkan  <br>Metal  <br>  <br>Might work on some OpenGL ES 2.0 devices, depending on driver extensions and features.|Everything in `2.5`, plus:  <br>`interpolators10 samplelod fragcoord`|
|`3.5`|Equivalent to [OpenGL ES 3.0](https://en.wikipedia.org/wiki/OpenGL_ES#OpenGL_ES_3.0).|DirectX 11 feature level 10+  <br>OpenGL 3.2+  <br>OpenGL ES 3+  <br>Vulkan  <br>Metal|Everything in `3.0`, plus:  <br>`interpolators15 mrt4 integers 2darray instancing`|
|`4.0`|Equivalent to [DirectX shader model 4.0](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-sm4), but without the requirement to support 8 MRTs.|DirectX 11 feature level 10+  <br>OpenGL 3.2+  <br>OpenGL ES 3.1+AEP  <br>Vulkan  <br>Metal (if no geometry stage is defined)|Everything in `3.5`, plus:  <br>`geometry`|
|`4.5`|Equivalent to [OpenGL ES 3.1](https://en.wikipedia.org/wiki/OpenGL_ES#OpenGL_ES_3.1).|DirectX 11 feature level 11+  <br>OpenGL 4.3+  <br>OpenGL ES 3.1  <br>Vulkan  <br>Metal|Everything in `3.5`, plus:  <br>`compute randomwrite msaatex`|
|`4.6`|Equivalent to [OpenGL 4.1](https://en.wikipedia.org/wiki/OpenGL#OpenGL_4.1).  <br>  <br>This is the highest OpenGL level supported on a Mac.|DirectX 11 feature level 11+  <br>OpenGL 4.1+  <br>OpenGL ES 3.1+AEP  <br>Vulkan  <br>Metal (if no geometry stage is defined, and no hull or domain stage is defined)|Everything in `4.0`, plus:  <br>`cubearray tesshw tessellation msaatex`|
|`5.0`|Equivalent to [DirectX shader model 5.0](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/d3d11-graphics-reference-sm5), but without the requirement to support 32 interpolators or cubemap arrays.|DirectX 11 feature level 11+  <br>OpenGL 4.3+  <br>OpenGL ES 3.1+AEP  <br>Vulkan  <br>Metal (if no geometry stage is defined, and no hull or domain stage is defined)|Everything in `4.0`, plus:  <br>`compute randomwrite msaatex tesshw tessellation`|
##### GPU features（#pragma require）的值
|**值**|**描述**|
|---|---|
|`interpolators10`|At least 10 vertex-to-fragment interpolators (“varyings”) are available.|
|`interpolators15`|At least 15 vertex-to-fragment interpolators (“varyings”) are available.  <br>  <br>**Note:** Internally, this also automatically adds `integers` to the list of requirements.|
|`interpolators32`|At least 32 vertex-to-fragment interpolators (“varyings”) are available.|
|`integers`|Integers are a supported data type, including bit/shift operations.  <br>  <br>**Note:** Internally, this also automatically adds `interpolators15` to the list of requirements.|
|`mrt4`|At least 4 render targets are supported.|
|`mrt8`|At least 8 render targets are supported.|
|`derivatives`|Pixel shader derivative instructions (ddx/ddy) are supported.|
|`samplelod`|Explicit texture LOD sampling (tex2Dlod / SampleLevel) is supported.|
|`fragcoord`|Pixel location (XY on screen, ZW depth in clip space) input in pixel shader is supported.|
|`2darray`|2D texture arrays are a supported data type.|
|`cubearray`|Cubemap arrays are a supported data type.|
|`instancing`|SV_InstanceID input system value is supported.|
|`geometry`|Geometry shader stages are supported.|
|`compute`|Compute shaders, structured buffers, and atomic operations are supported.|
|`randomwrite` or `uav`|“Random write” (UAV) textures are supported.|
|`tesshw`|Hardware tessellation is supported, but not necessarily tessellation (hull/domain) shader stages. For example, Metal supports tessellation, but not hull or domain stages.|
|`tessellation`|Tessellation (hull/domain) shader stages are supported.|
|`msaatex`|The ability to access multi-sampled textures (Texture2DMS in HLSL) is supported.|
|`sparsetex`|Sparse textures with residency info (“Tier2” support in DirectX terms; `CheckAccessFullyMapped` HLSL function).|
|`framebufferfetch` or `fbfetch`|Framebuffer fetch (the ability to read input pixel color in the pixel shader) is supported.|
|`setrtarrayindexfromanyshader`|Setting the render target array index from any shader stage (not just the geometry shader stage) is supported.|

### HLSL的目标图形API和运行平台

#### 包含或排除API
By default, Unity compiles all shader programs for each graphics API in the list for the current build target. Sometimes, you might want to compile certain shader programs only for certain graphics APIs; for example, if you use features that are not supported on all platforms.

To compile a shader program only for given APIs, use the `#pragma only_renderers` directive. You can pass multiple values, space delimited.

This example demonstrates how to compile shaders only for Metal and Vulkan:

```
#pragma only_renderers metal vulkan
```

To exclude shader code from compilation by given compilers, use the `#pragma exclude_renderers` directive. You can pass multiple values, space delimited.

This example demonstrates how to exclude a shader from compilation for Metal and Vulkan:

```
#pragma exclude_renderers metal vulkan
```

#### 为图形API生成分等级变体
内置管线，一般URP中并不需要你进行设置。

#### Unity支持的图形API
|**值**|**描述**|
|---|---|
|`d3d11`|DirectX 11 feature level 10 and above, DirectX 12|
|`gles`|OpenGL ES 2.0, WebGL 1.0|
|`gles3`|OpenGL ES 3.x, WebGL 2.0|
|`ps4`|PlayStation 4|
|`xboxone`|Xbox One and GameCore, DirectX 11 and DirectX 12|
|`metal`|iOS/Mac Metal|
|`glcore`|OpenGL 3.x, OpenGL 4.x|
|`vulkan`|Vulkan|
|`switch`|Nintendo Switch|
|`ps5`|PlayStation 5|

### 在HLSL中声明并且使用着色器关键词
#### 用pragma声明shader keywords
你可以使用以下的：

|**Shader directive**|**Branching type**|**Shader variants Unity creates**|
|---|---|---|
|`shader_feature`|[Static branching](https://docs.unity.cn/cn/2022.3/Manual/shader-branching.html#static-branching)|Variants for keyword combinations you enable at build time|
|`multi_compile`|Static branching|Variants for every possible combination of keywords|
|`dynamic_branch`|[Dynamic branching](https://docs.unity.cn/cn/2022.3/Manual/shader-branching.html#dynamic-branching)|No variants|
Read more about [when to use which shader directive](https://docs.unity.cn/cn/2022.3/Manual/shader-conditionals.html).
See [shader keyword limits](https://docs.unity.cn/cn/2022.3/Manual/shader-keywords.html#keyword-limits).

#### keywords set如何工作
在一共#pragma声明里面可以有多个keywords，这些keywords组起来成为一共keywords set。
比如，你可以在这个声明中使用三个keywords:

```
#pragma shader_feature REFLECTION_TYPE1 REFLECTION_TYPE2 REFLECTION_TYPE3
```

你也可以在一个shader中声明多个集.。比如下面就是两个:

```
#pragma shader_feature REFLECTION_TYPE1 REFLECTION_TYPE2 REFLECTION_TYPE3
#pragma shader_feature RED GREEN BLUE WHITE
```
#### 让shader根据条件工作：

你可以在shader中通过关键字，让Pass中的代码块根据你想要的条件工作：
```
#pragma multi_compile QUALITY_LOW QUALITY_MED QUALITY_HIGH

if (QUALITY_LOW)
{
    // code for low quality setting
}
```
Unity如何处理你的shader代码和你使用的指令有关：
如果你使用的是dynamic_branch，那么我建议你不要用它
如果你使用的是shader_feature或者multi_compile，那么untiy会为每一个变体生成一个shader variant。当你启用一个shader variant的时候，unity会把这个变量对应的着色器发送给GPU。这也叫静态分支（static branching）。

但是要注意的是：
千万不要滥用multi_compile这一个指令，他会给存储带来很大的负担。
###### 1. `#pragma shader_feature` （推荐用于材质属性）

- **用途**：声明一些**仅在编辑器中启用**的关键字，通常与材质的公开属性（如 `[Toggle]`, `[Enum]`）联动。
- **变体生成策略**：**精益生产**。Unity 只会为那些**在项目中被材质实际使用到的关键字组合**生成着色器变体，并最终打包到游戏中。
- **优点**：非常高效，能有效控制最终游戏的包体大小和内存占用。
- **示例**：
    // 在着色器中声明一个功能，用于切换反射类型
    #pragma shader_feature REFLECTION_TYPE1 REFLECTION_TYPE2 REFLECTION_TYPE3
    
    // 在Properties块中链接到材质属性，方便美术师操作
    Properties {
        [Enum(Type1, Type2, Type3)] _ReflectionType ("Reflection Type", Float) = 0
    }
    
    **结果**：如果你的项目中只有材质使用了 `REFLECTION_TYPE1` 和 `REFLECTION_TYPE3`，那么 `REFLECTION_TYPE2` 对应的变体**根本不会被打包**。

###### 2. `#pragma multi_compile` （用于全局和必需功能）
- **用途**：声明一些**必须始终可用**的关键字，无论是否有材质用到它。最常见的就是处理光照和阴影。
- **变体生成策略**：** brute force（暴力生成）**。Unity 会为**所有可能的关键字组合**生成着色器变体。
- **缺点**：会急剧增加变体数量（组合爆炸），导致编译时间变长、游戏包体变大、运行时内存占用增高。
- **示例**：
    // 这是Unity内置管线中处理光照的典型代码，它会生成海量变体
    #pragma multi_compile DIRECTIONAL DIRECTIONAL_COOKIE POINT POINT_COOKIE SPOT
    #pragma multi_compile SHADOWS_DEPTH SHADOWS_CUBE SHADOWS_SOFT
    
    **结果**：上面两行指令，假设第一行有5种可能，第二行有3种可能，它们会组合出 5 x 3 = **15 个变体**，并且**所有这些变体都会被完整地打包到游戏中**。


##### make keywords local
默认的keyword都是全局的，#pragma shader_feature_local能

##### shader variant for disabled keywords
无论是#pragma multi_compile还是#pragma shader_feature都适用的规则：
在只有一个keyword的时候，会为这个keyword开启和关闭都设置variant；
在有多个keyword的时候，会每一个keyword都设置开启关闭的variant，如果希望每一个keyword都不被开启，可以在前面加一个 _ 表示全部都不采用的情况。

##### 创建keyowrds的快捷指令
在unity文档的官网里面使用的都是内置渲染管线的快捷指令，这些在URP中就失效了。



