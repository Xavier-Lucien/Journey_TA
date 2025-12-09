UE的渲染系统是UE源码中的一部分。UE最核心的源码位置都在Engine\Source\Runtime这个目录下，渲染系统知识UE的整个Engine中的一个模块，或者说是一个比较重要的模块。游戏引擎的构成很复杂，物理、声音等等都是其中的模块。
## 帧生命周期与游戏线程：

游戏的帧逻辑和渲染逻辑并不是完全同步的，而是一个密度很高的流水线：
```
Time: 0ms ────────> 16ms ────────> 32ms 
	  │              │              │ 
Game: [帧N逻辑]   [帧N+1逻辑]     [帧N+2逻辑] 
        │            │              │ 
Render: └─[帧N-1渲染] └─[帧N渲染]     └─[帧N+1渲染] 
                  │            │ 
RHI:              └─[帧N-2提交] └─[帧N-1提交]
```
理解游戏中的渲染系统，大致可以从3个线程看，分别是Game Thread、Render Thread、RHI Thread。现在看Render Thread和RHI Thread其实是比较懵的，Render Thread可以看成是走渲染管线流程，RHI Thread则是翻译你要干的渲染的活，结合图形API转化为实际的GPU 程序。
事实上，如果你打开Engine\Source\Runtime这个文件夹以后，你能看见的很渲染相关的文件夹是：RenderCore、Renderer、RHI、RHICore、D3D12RHI、OpenGLDrv、VulkanRHI就很多，而另外还有TextueUtilitiesCommon，MeshConversion这些文件夹。这里先提前介绍一下，RenderCore大致是用来 管理渲染资源，生成RHICommandLists的，Renderer可以理解为相机走完经典的渲染管线（剔除，渲染、后处理）的一个东西。RHI是在图形API上一次的一个抽象，RHI可以对不同类型的API做适配。RHI以后就到GPU的程序了，现在GPU上着色器都是HLSL之类的，相对固定，在Engine\Shaders里面都是自带的shader文件。
但是如果从总的全局来看，GameLoop就是整个游戏的心脏，GameLoop出发渲染线程。由于渲染时需要时间的，这个延迟通常时在1-2帧之内。

### 场景数据流：

游戏渲染过程中，整个数据的流动方向是：游戏世界 → Scene Proxy(一个容器) → 渲染命令 → RHI → GPU。渲染命令里面比较复杂，这个是[[UE里面的Renderer]]模块专门的内容。
```
游戏世界 (AActor/UPrimitiveComponent)
    ↓
Scene Proxy (FPrimitiveSceneProxy)
    ↓
FScene (场景数据管理)
    ↓
┌─────────────────────────────────────┐
│  IRendererModule::BeginRenderingViewFamily  │ ← Renderer 入口
│        ↓                              │
│  FSceneRenderer::Render()            │ ← Renderer 核心
│    ├─ InitViews()                    │
│    ├─ ComputeViewVisibility()        │
│    ├─ FMeshPassProcessor             │
│    │     → FMeshDrawCommand          │
│    └─ RenderPasses (via RDG)         │
└─────────────────────────────────────┘
    ↓
FRDGBuilder::Execute()
    ↓
FRHICommandList (RHI 命令)
    ↓
GPU
```
如果结合线程来看就是：
```
┌─────────────────────────────────────────────────────────────────┐
│  【游戏线程】                                                     │
├─────────────────────────────────────────────────────────────────┤
│  AActor/UPrimitiveComponent                                     │
│      ↓ MarkRenderStateDirty()                                   │
│  FPrimitiveSceneProxy (渲染数据代理)                              │
│      ↓ FScene::AddPrimitive()                                   │
│  FScene (场景数据容器)                                            │
│      ↓                                                          │
│  【关键入口 1】IRendererModule::BeginRenderingViewFamily()        │
│      │                                                          │
│      └─ 创建 FSceneRenderer (Deferred/Mobile)                    │
│      └─ ENQUEUE_RENDER_COMMAND() 切换到渲染线程                   │
└─────────────────────────────────────────────────────────────────┘
                        ↓↓↓ 线程切换 ↓↓↓
┌─────────────────────────────────────────────────────────────────┐
│  【渲染线程 - Renderer 模块】                                     │
├─────────────────────────────────────────────────────────────────┤
│  【核心类】FSceneRenderer (抽象基类)                               │
│             ├─ FDeferredShadingSceneRenderer (延迟渲染)          │
│             └─ FMobileSceneRenderer (移动端)                     │
│      ↓                                                          │
│  【阶段 1】FSceneRenderer::InitViews()                           │
│      ├─ 计算视图矩阵                                              │
│      ├─ 创建 ViewUniformBuffer                                   │
│      └─ 初始化 GPU Scene                                         │
│      ↓                                                          │
│  【阶段 2】FSceneRenderer::ComputeViewVisibility()               │
│      ├─ 视锥剔除 (Frustum Culling)                               │
│      ├─ 遮挡剔除 (Occlusion Culling via HZB)                     │
│      ├─ 距离剔除 (Distance Culling)                              │
│      └─ 输出: VisiblePrimitives 列表                             │
│      ↓                                                          │
│  【阶段 3】FMeshPassProcessor::AddMeshBatch()                    │
│      ├─ 获取材质 Shader                                          │
│      ├─ 创建 Pipeline State                                     │
│      ├─ 绑定 Shader 参数                                         │
│      └─ 输出: FMeshDrawCommand                                  │
│      ↓                                                          │
│  【阶段 4】FDeferredShadingSceneRenderer::Render(FRDGBuilder&)   │
│      ├─ RenderPrePass(GraphBuilder, ...)                        │
│      ├─ RenderBasePass(GraphBuilder, ...)                       │
│      ├─ RenderLights(GraphBuilder, ...)                         │
│      ├─ RenderTranslucency(GraphBuilder, ...)                   │
│      └─ AddPostProcessingPasses(GraphBuilder, ...)              │
│      ↓                                                          │
│  【阶段 5】FRDGBuilder::Execute()                                │
│      (RDG 生成 RHI 命令)                                         │
└─────────────────────────────────────────────────────────────────┘
                        ↓↓↓
┌─────────────────────────────────────────────────────────────────┐
│  【RHI 线程】                                                    │
├─────────────────────────────────────────────────────────────────┤
│  FRHICommandList::DrawIndexedPrimitive()                        │
│      ↓                                                          │
│  平台 RHI (D3D12/Vulkan/Metal)                                   │
│      ↓                                                          │
│  GPU 执行                                                        │
└─────────────────────────────────────────────────────────────────┘
```