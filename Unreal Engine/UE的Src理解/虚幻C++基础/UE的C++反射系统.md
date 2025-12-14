## 对象
虚幻引擎中所有对象的基类都是UObject。UCLASS宏的作用就是标记UObject的子类，以便UObject处理系统可以识别它们。
### UCLASS宏：
UCLASS宏为UObject提供了一个UCLASS引用，用于描述它在虚幻引擎中的类型。每个UCLASS都保留一个称作Class of Default Object的对象。CDO本是是是一个默认模板对象。


### UObject提供的功能
垃圾回收
引用更新
反射
序列化
默认属性变换自动更新
自动属性初始化
自动编辑器整合
运行时类型信息可用
网络复制

### 虚幻头文件工具：
利用UObject派生类型所提供的功能