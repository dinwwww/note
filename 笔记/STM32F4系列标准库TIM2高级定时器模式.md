# STM32F4系列标准库TIM2高级定时器模式

## 编码器正反转

![image-20240826142537404](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240826142537404.png)

## 编码器模式结构体关键语句

TIM_ICInitstructure.TIM_Channel = TIM_Channel_1;//配置IC通道

TIM_ICInitstructure.TIM_ICPolarity =TIM_ICPolarity_Rising;//极形是否反转，选择不反转极形

TIM_EncoderInterfaceConfig(TIM2,TIM_EncoderMode_TI12,TIM_ICPolarity_Falling,TIM_ICPolarity_Falling);//配置编码器模式

TIM_ICPolarity_Falling配置的是相反信号的电平

正常情况如图

![image-20240826145729205](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240826145729205.png)

两个都是取反对应电平信号，那么就是A相信号在前面，TI1相信号先上升沿，对应TI2就是低电平那么计次+1

轮到TI2上升沿，对应电平取反对应此时TI1是高电平那么计次+1

轮到TI1下降沿，对应电平取反此时TI2是高电平那么计次+1

轮到TI2下降沿，对应电平取反此时TI1是低电平那么计次+1

