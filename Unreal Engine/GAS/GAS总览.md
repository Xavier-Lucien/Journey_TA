
## Game Ability是什么
Game ability是Actor可以拥有并且重复出发的action。设计的时候主要为了应对动作这个系统的复杂度和逻辑关系。game ability本身是一种抽象。出于三个考虑：
### 追踪Ability拥有者：
GA需要有归属。GA在执行并且运算以后需要知道它的拥有者并且使用这个actor上面的属性。

### 追踪Ability的状态：
ability需要追踪什么时候被激活，过程中和终止并且不激活

### 同步Ability的执行：
主要是蓝图通信上。Game Ability能与多个系统互动。
- Activating animation montages.
    
- Taking temporary control of a character's movement.
    
- Triggering visual effects.
    
- Performing overlap or collision events.
    
- Changing characters' stats, either temporarily or permanently.
    
- Increasing or decreasing in-game resources.
    
- Allowing or blocking the activation of other abilities.
    
- Handling cooldowns to restrict ability usage.
    
- Getting interrupted by in-game events.
    
- Canceling other abilities in-progress.
    
- Making major state changes to a character, such as activating a new movement mode.
    
- Responding to input in the middle of other interactions.
    
- Updating UI elements to show in-game status for abilities.