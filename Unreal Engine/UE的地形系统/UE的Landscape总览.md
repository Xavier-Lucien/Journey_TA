
## UE的Landscape类的继承关系

```
ALandscape (整体地形Actor)  
└── ALandscapeProxy (地形代理,支持流式加载)  
    └── ULandscapeComponent (组件,最小渲染单元)  
        ├── NumSubsections (子区域数量,通常1或2)  
        ├── SubsectionSizeQuads (每个子区域的四边形数)  
        └── ULandscapeHeightfieldCollisionComponent (碰撞组件)
```


## World Partition：

```
1. 世界被自动划分为 Grid Cells  
   └─> 每个 Cell 包含若干 LandscapeStreamingProxy  
  
2. 玩家移动触发流式加载  
   └─> World Partition 系统计算可见 Cells  
       └─> 异步加载所需的 LandscapeComponent  
           └─> RVT 系统渲染该区域的虚拟纹理  
               └─> Nanite 流式加载几何数据(UE5.1+)  
  
3. 远离区域自动卸载  
   └─> 释放内存和显存
```
