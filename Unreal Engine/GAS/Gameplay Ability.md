能力除了技能以外还有很多。攻击是能力，被打也是能力。除了基础移动，射线检测和UI这些几乎都可以算是Gameplay ability。

GameAbility是游戏逻辑的主要书写的地方。

Game Ability可以重载：ActivateAbility、CommitAbility、CancelAbility、EndAbility
![[Pasted image 20251216140439.png]]

拥有[[Ability System Component]]的Actor通过调用TryActivateAbility触发。
每一个GA在被激活以后都会变成一个FGameAbilitySpec的对象。