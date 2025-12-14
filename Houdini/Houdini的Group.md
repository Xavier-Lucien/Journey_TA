
## Houdini里Group Create的面板：

![[Pasted image 20251214120942.png]]

Houdini的Group和Group Create是两个一样的节点，可以分为Primitive，Point，Edge，Vertex这么几个Group Type。
对于同名的组，你可以设置，Replace Existing，Union with Existing，Intersect with Existing，Subtract from Existing。
Base Group打勾是新建组并且全选， 不打勾是新建组但是全不选（全部赋值为0或全部赋值为1）。根据Group Type进行选择，右边的小鼠标箭头可以手动选择。
这个节点主要是为了做以上这么几个种类的东西的自动化选择，当然你还可以手动选择。这个可以看成是给geometry spread sheet里面的东西添加group的二值标签

Group Copy这个方法是要针对两个结构完全一样的Group把其中的一个Group复制到另一个中。这个节点的效果相当于在Geometry Spread Sheet里面添加了一列。

Group Transfer是按照位置来计算的，算法是kdtree。把reference的组里面的Group名称按照距离（这个节点上面的distance threshold按钮）来对source里面的与reference组里Group Type相同的东西进行筛选。在spread sheet里面没有任何的影响，就是本身是一个kdtree的算法。

## Houdini的设计理念：

和C++里面很像，实现代表深拷贝，虚线代表const引用。核心是spread sheet。
整个工作流可以看成是在堆上面不断创建新的拷贝对象，以及进行对拷贝对象进行attribute的增改删查。