渲染管线主要分成三种：内置渲染管线、[[URP]]、HDRP
内置渲染管线基本上不用了，是因为它相对写死。后来Unity保留了C++源码编译的核心而暴露了更多的C#的API，更新了SRP，再后来有了URP和HDRP。URP比较通用，可以在移动端和桌面端使用。HDRP基本上是在端游使用。基本上现在技术美术都在学习URP。

首先区分一下Graphics Pipeline和Render Pipeline的区别。我觉得下面这张图可能叫做Graphics pipline更合适，比较粗糙。但是这是最基础的。实际回归到本质上，你再Unity里面使用的内置渲染管线、URP、HDRP这些渲染管线也都是按照这个逻辑组装起来的。但是仅仅了解这些应该还是不太够用的。
你要知道的是，无论是哪一种渲染管线，都是再一帧（frame）里面运算结束的，游戏通常都要保证再60帧左右，是不是感觉这个计算速度很快，所以我们技术美术在工作的时候，就像是开了慢动作一样对每一帧的渲染逻辑进行设计。

![](https://docs.unity.cn/cn/2022.3/uploads/Main/BestPracticeLightingPipeline3.svg)

怎么处理光照？
首先要选择一个渲染管线。然后决定如何产生间接光照，并相应地选择一个全局光照系统。确保为您的项目适当地调整了所有全局光照设置之后，您可以继续添加[光源](https://docs.unity.cn/Manual/Lighting.html)、[发光表面](https://docs.unity.cn/Manual/StandardShaderMaterialParameterEmission.html)、[反射探针](https://docs.unity.cn/Manual/class-ReflectionProbe.html)、[光照探针](https://docs.unity.cn/Manual/LightProbes.html)和[光照探针代理体 (LPPV)](https://docs.unity.cn/Manual/class-LightProbeProxyVolume.html)。所有这些光照对象的用法和特性的详细介绍超出了本文的范围，因此，建议您阅读手册中的“光照”部分，以学习如何在项目中正确地使用光照。
下面是光照管线：

![](https://docs.unity.cn/cn/2022.3/uploads/Main/BestPracticeLightingPipeline15.svg)

## 渲染管线
### 内置渲染管线
2018年以前，只有一个渲染管线：内置渲染管线，这东西现在不太够看了。略

### HDRP

HDRP主要是用在主机游戏上面，这里面比较好的技术是对前向渲染和延迟渲染提供了瓦片和聚类的渲染。这在进行多光源光照的时候有妙用。

![](https://docs.unity.cn/cn/2022.3/uploads/Main/BestPracticeLightingPipeline6.svg)

瓦片是帧的一个小型二维方形像素部分，而聚类则是摄像机视锥体中的一个三维体积。瓦片和聚类渲染技术都依赖于影响每个瓦片和聚类的光源的列表，然后可以用相应的已知光源列表在一个通道中计算其光照。不透明对象很可能使用瓦片系统进行着色，而透明对象则依赖于聚类系统。该渲染器的主要优点是，与内置渲染管线（延迟）相比，光照处理速度更快，带宽消耗也大大减少，因为内置渲染管线依赖于更慢的多通道光照积累。

### URP
urp相比于其它的两个管线就很全能了。URP可定制性比较高，能在低端设备上面跑，如果你自己厉害也可以在桌面端达到很惊人的画质效果。
Unity官方文档把它介绍为"...a fast single-pass forward renderer."。这里面有几个关键词：单通道、前向渲染。
- **前向渲染**：这是一种处理光和物体关系的经典方法。您可以想象一个画家在画布上作画：他先画好所有物体的形状和颜色（**基础Pass**），然后再一笔一笔地为这些物体加上光照效果（**额外的逐光Pass**）。物体受的光照越多，需要画的“笔数”就越多，就越耗时。
-  **单通道**：这是 URP 的魔法所在！它对这个传统方法进行了超级优化。URP 的画家非常厉害，他**在一次作画中，就同时处理所有光照信息**。对于每个物体，他看一眼场景里所有影响它的光，然后一次性把光和物体的颜色都画好。
那这个到底是怎么实现的呢？
URP会对每一个物体进行灯光剔除。Unity不会再白场景中的所有光的信息都塞给shader，而是会为每一个要花的物体做一次计算：
1.检查这个物体的位置和范围
2.找出能照亮这个物体的光源
3.只把这些相关的光源信息打包，然后一次性发送给Shader。

half4 frag(Varyings IN) : SV_Target {
    // ... 准备表面数据 surfaceData 和输入数据 inputData ...

    // 【第1步：计算主光源贡献 - 必须且有特殊通道】
    Light mainLight = GetMainLight();
    // 还经常需要为主光源计算阴影
    half4 shadowCoord = TransformWorldToShadowCoord(IN.positionWS);
    mainLight.shadowAttenuation = MainLightRealtimeShadow(shadowCoord); 
    
    half3 color = LightingLambert(mainLight, surfaceData.normal, surfaceData.albedo);

    // 【第2步：计算附加光源贡献 - 可选且受数量限制】
    int additionalLightsCount = GetAdditionalLightsCount();
    for (int i =  0; i < additionalLightsCount; ++i) {
        // 注意：这里需要传入世界坐标，用于计算每个光源的衰减
        Light addLight = GetAdditionalLight(i, IN.positionWS); 
        // 累加每一个附加光源的贡献
        color += LightingLambert(addLight, surfaceData.normal, surfaceData.albedo);
    }

    // ... 最后组合颜色 ...
    return half4(color, 1.0);
}
可以看到这是一个大概的代码框架。当然看过冯乐乐的《UnityShader入门精要》这本书就会知道，GPU在做条件判断和循环（每次循环都要做一下条件判断）是性能会骤降，可以在移动端把这些additionallight给停掉。

### 全局光照系统
全局光照系统在Untiy里面大致可以分为两种：
#### 实时全局光照
技术：
借助Enlighten中间件
工作原理：
1.预计算：Unity会先进行一个预计算的过程。这不是烘焙，它不生成lightmap，而是计算场景的几何信息如何反弹光线，并且生成一套用于实时解算的数据结构。
2.实时运行：在游戏运行时，Enlighten 系统会**每一帧**都根据这些预计算数据和当前场景中的直接光照（如主灯开关、颜色变化），来动态计算间接光照的效果。
**核心特点**：
**动态**：你可以实时改变光源的颜色、强度，甚至开关灯，间接光照也会随之实时变化，产生非常动态的效果。
**性能开销**：运行时需要一定的 CPU 和 GPU 开销来进行实时计算。
**动态物体参与GI**：如果游戏对象勾选了 **`ContributeGI`** 选项，并且它在运行时移动（比如一个红色的球），它就能将自身的颜色反射到周围的环境上，实现动态的色溢（Color Bleeding）效果。
**适用场景**：主要用于需要光源发生**实时变化**的场景，如昼夜循环、可开关的室内灯光、动态改变颜色的氛围灯等。
#### 烘焙全局光照
这个系统是当前的主流的选择，将计算结果直接烘焙到各种数据中，运行的时候直接读取。
![](https://docs.unity.cn/cn/2022.3/uploads/Main/BestPracticeLightingPipeline4.svg)

#### 静态与动态全局光照的对比
无论使用哪种全局光照系统，在烘焙/预计算光照过程中，Unity 只会考虑标记为[“Contribute GI”](https://docs.unity.cn/Manual/StaticObjects.html)的对象。动态（即非静态）对象必须依赖于放置在整个场景中的光照探针来获得间接光照。
由于光照的烘焙/预计算是一个相对缓慢的过程，所以只有具有不同光照变化（例如凹度和自我阴影）的大型和复杂资源才应被标记为“Contribute GI”。获得均匀光照的较小和凸面网格不应进行这样的标记，因此它们应该从[光照探针](https://docs.unity.cn/Manual/LightProbes.html)获得间接光照；光照探针中存储了更简单的光照近似值。较大的动态对象可以依赖 [LPPV](https://docs.unity.cn/Manual/class-LightProbeProxyVolume.html)，以便获得更好的本地化间接光照。要最大程度减少烘焙时间并同时保持足够的光照质量，最重要的就是限制场景中标记为“Contribute GI”的对象数量。可在本[教程](https://learn.u3d.cn/topics/graphics/introduction-precomputed-realtime-gi?playlist=17102)中详细了解这个优化过程和探针光照的重要性。