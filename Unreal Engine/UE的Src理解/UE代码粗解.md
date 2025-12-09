Unreal Engine的代码书写规范是匈牙利命名法，这能极大提速对虚幻引擎的理解：
#### F前缀：普通的C++类：

```
// ✅ F = Plain C++ struct/class (不继承UObject) 
class FMaterial // 材质的运行时表示（非UObject） 
class FRendererModule // 渲染器模块（非UObject） 
class FSceneView // 场景视图（非UObject） 
class FMeshBatch // 网格批次（非UObject） 
struct FViewMatrices // 视图矩阵（struct） 
struct FSceneTextures // 场景纹理（struct） 
// 特点： 
// ✅ 可以在栈上创建（不需要new） 
// ✅ 没有垃圾回收 
// ✅ 性能更高（没有反射开销） 
// ✅ 适合作为临时数据、数学类型、渲染数据等 
//常见用法：创建在栈上 FVector MyVector(1.0f, 2.0f, 3.0f); 
// ✅ 栈对象 FMeshBatch MeshBatch; 
// ✅ 栈对象 
// 不需要垃圾回收 
void SomeFunction() 
{ 
	FSceneView View; 
	// 自动分配 // ... 
} 
// 自动析构，无需手动管理
```

U