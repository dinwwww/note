# STM32高级定时器互补PWM生成与死区控制带刹车调试配置过程

## 死区时间

![image-20240920151548443](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920151548443.png)

在这个过程中，采样时间tDTS= tCK_INT 时间的公式转换成频率的公式 fDTS=fCK_INT/2 

也就是1分频 2分频 4分频

![image-20240920151854164](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920151854164.png)

判断DTG[7:5],选择计算公式

![image-20240920152345039](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920152345039.png)

![image-20240920152320433](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920152320433.png)

判断前三位选择计算公式

第五位是公式里面的一些系数

## 刹车断路功能

![image-20240920152629129](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920152629129.png)

TIMx_BKIN是固有的引脚做复用确认为刹车功能，要使用复用模式，且要根据刹车引脚选择有效电平来选择上拉电阻还是下拉电阻



![image-20240920152716787](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920152716787.png)

选择使能刹车功能后，可以根据寄存器选择生成固定的电平，且互补pwm生成不会同时处于有效电平

![image-20240920152911458](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920152911458.png)

当发生刹车后，PWM互补通道为空闲模式

<img src="C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920153658071.png" alt="image-20240920153658071" style="zoom:150%;" />

  TIM_OCInitStructure.TIM_OCIdleState=TIM_OCIdleState_Reset;

  TIM_OCInitStructure.TIM_OCNIdleState=TIM_OCNIdleState_Reset;

具体是这两个参数配置

## 配置互补通道

![image-20240920160031273](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920160031273.png)

因此本处采用PWM模式1 主输出通道有效电平为高电平，主通道的占空比是30%，互补通道会自动取反

![image-20240920160107513](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920160107513.png)

空闲状态的取值

![image-20240920160455332](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920160455332.png)

死区配置时，空闲模式与运行模式下的空闲状态

暂时用不到

![image-20240920160823821](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920160823821.png)

![image-20240920160634123](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920160634123.png)

![image-20240920160619981](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920160619981.png)

在刹车模式下，不能同时设置刹车之后的电平为有效电平，例如，本配置有效电平是高电平。在刹车之后都都是高电平即有效电平，因此硬件会保护不会同时产生有效电平。所以为错误配置

![image-20240920162150750](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920162150750.png)

## 存疑

目前，配置好这两个空闲状态，不管如何操作，是高电平低电平，都是由引脚复用输出状态决定，取决于是否推挽输出，开漏输出有无上下拉电阻。待解决（要使得互补PWM输出，又必须设置复用推挽输出，因此存疑）

![image-20240920165908827](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240920165908827.png)