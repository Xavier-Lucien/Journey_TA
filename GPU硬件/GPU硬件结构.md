## GPU结构总览

英伟达画图这么画：
- 红色 / 绿色 → 可编程单元（SM / Tensor / ALU）
    
- 蓝色 → 图形前端 / 固定功能逻辑单元（Rasterizer 前端、Primitive Setup、Triangle Dispatch）
    
- 黄色 / 橙色 → 存储如L2 / Memory Controller / HBM 接口

![[3060 arch.jpg]]![[ga100-full-gpu-128-sms.png]]


### GPU的计算层级

整个的层级应该是从GPU→GPC→TPC→SM→warp→线程。上面的这个GA100的图里面其实可以看见，整张图是一个GPU，然后下面是8个GPC，每个GPC里面有8个TPC，每个TPC里面有2个SM（但是需要注意的是，我说的是GA100和“A100的一张理想满血架构图”，实际上的A100其实是只有7个GPC）。而在3060里面有3个GPC，每个GPC里面有5个TPC，每个TPC有2个SM（3060laptop有30个SM，而桌面端的只有28个，很罕见）。

#### GPU整体
包括：所有 GPC、全局 L2 Cache、显存控制器（HBM/GDDR）、PCIe / NVLink、ROP、Copy engines。GPC是下一个层级的抽象，我们暂且不管。L2 Cache是在GPU内部的存储，相对能访问速度较快，显存控制器就是负责去VRAM读取数据的，PCIe是连接CPU和GPU的，NVLink是英伟达发明的显卡串烧技术，ROP和Copy Engines在图里面没看见，是比较功能性的器件，ROP（Raster operator）不在 GPC 内，它位于“内存/L2 分区”（memory partition）一侧，和 L2 cache、内存控制器同属顶层全局结构。通常是多个分区的 ROP 并行工作，而不是单一全局 ROP。片段在完成像素着色后会根据目标帧缓冲区域被路由到对应的 ROP 分区，进行颜色混合、深度/模板测试、写回等；从功能上看它服务于所有 GPC，但实现是分区化的。在 GPC 之外，属 GPU 全局 I/O/DMA 引擎。通常有多个 Copy Engine（如 CE0/CE1），支持异步 H↔D、D↔D、P2P（NVLink/PCIe）拷贝与 memset。这里其实注意一下ROP就行了，Stencil Test、Depth Test这些功能都需要ROP执行的。