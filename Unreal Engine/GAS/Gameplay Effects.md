[[Gameplay Attribute]]的改变都需要通过Gameplay Effect来做。但是GE并不只是修改属性，所有的技能的效果都是用GE来表示的，包括逻辑效果（伤害，扣血），给对方标签（达成眩晕，沉默），基于对方能力，触发特效效果[[Gameplay Cue]]

Gamplay Effect是一个纯蓝图配置的，不需要重载来处理。GE配置：类型，修改器，周期，应用需求，溢出处理，过期处理，显示处理，Tags条件，免疫，堆叠，能力赋予

一定要注意Attribute Set只能通过GE来修改。

从源码来看，GE由GA应用，创造一个FGameplayEffectSpec，这类似一个数据模板形成一个实例，然后激活GE，放入容器并且最后存到[[Ability System Component]]。
![[Pasted image 20251216133830.png]]