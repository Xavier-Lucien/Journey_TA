Unity shader是要附着在材质上面配合使用提升画面效果的。
## Unity ShaderL结构：
Shader "shaderName"{
Properties{
//shader的参数值
}
SubShader{
//显卡A使用的子着色器
}
SubShader{
//显卡B使用的子着色器
}
Fallback "VertexLit"
}

### SubShader：
可以包含多个针对不同的显卡型号使用，但是最少要有一个。
SubShader{
	//可选
	[Tags]
	//可选
	[RenderSetup]
	Pass{
	}
	//Other Passes（可选）
}
[RenderSetup]状态和[Tags]标签的设置可以在SubShader里面也可以在Pass里面。在SubShader里面的话就会直接应用在每一个Pass上面。
应该尽可能的减少Pass的数量。

#### [RenderSetup]状态设置：
Cull Back| Front| Off
ZTest Less| Greater| LEqual| GEqual| NotEqual| Always
ZWrite On| Off
Blend SrcFactor| DstFactor

#### [Tags]标签设置：
是键值对：形如
Tags {"TagName1" = "Value1"  "TagName2"="Value2}
标签类型有很多，比如说Queue、RenderType

#### Pass语义块：
Pass{
	[Name]
	[Tags]
	[RenderSetup]
}
可以给pass进行命名，
Name "MyPassName"
别的Pass可以通过使用名称进行调用
UsePass "MyShader/MyPassName"
Pass里面也可以有标签：不同于SubShader的标签，这些标签告诉渲染引擎怎么样渲染物体Tags{"LightMode" = "ForwardBase"}
