# SVPWM调试注入通道与高级定时器三路互补PWM，第四路触发注入通道转换心得

调试SVPWM的时候，由于复用模式没有更改GPIO引脚，导致TIM1 复用在PA上面，导致现象就是一烧录程序就无法读取到芯片，需要长按复位按键，且用调试软件把单片机存储扇区全部擦除，查看扇区也变成乱码

![image-20241025141833896](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025141833896.png)

上下桥臂未打开正确复用功能，查看数据手册，恰好复用在无法使用的引脚上，猜测这部分寄存器值是被单片机保护的，但是标准库在代码上没有验证是否可以写入，因此就直接改动这部分寄存器，导致单片机无法读取芯片，存储扇区值全部乱掉，这个引脚是13 14引脚，是给烧录口使用的，猜测原因

![image-20241025141906294](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025141906294.png)

未做保护

![image-20241025142130671](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025142130671.png)

调试ADC通道时，此部分配置适用于规则组，未选择触发源，就是只能用软件触发

![image-20241025142210856](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025142210856.png)

![image-20241025142301553](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025142301553.png)

要使得注入获取外部信号来触发，需要单独设置，并且采样时间不是越短越好，因为不同采样电阻与运放电路需要一定时间来完成自身释放，因此这个采样间隔与时间是一个值得讲究的选取与计算

![image-20241025142351179](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025142351179.png)

在调试PWM CH4通道的时候，关闭CH4的PWM互补通道生成

![image-20241025142612548](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025142612548.png)

并且这个CH4的CCR4选取很有深度，因为这个PWM模式是中央对齐，且PWM模式1，CNT小于CCR4的时候是高电平有效，但是此时不会产生CC4事件来触发ADC注入通道，要使得比较事件后，改变极性那么就是低电平，又由于是中央对齐模式，那么会改变回高电平，此时产生CC4上升沿的比较事件，那么就会产生ADC采样注入通道，且，此时刚好是三路PWM上半PWM全是低电平时刻，那么下半桥就为高电平，此时采样最为准确，因为中央对齐模式PWM其实是高低高如此排布，但是在整个运行周期可以看成低高低模式。

![image-20241025142656785](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025142656785.png)



如图分析，绿色ADC中断翻转电平，每次翻转时刻都是在上半桥关闭时刻，因此在整个周期上来看 可以看成PWM刚开始的时候，其实时实际产生波形高低高 仔细分析-。-，需要完全对应上，就需要改中央对齐模式或者PWM模式

![28b7f299d5f7cc3baf9ee349c1d9df2](C:\Users\ding\Documents\WeChat Files\wxid_u4pvjwgc5whb22\FileStorage\Temp\28b7f299d5f7cc3baf9ee349c1d9df2.jpg)

互补通道的设置，开启互补通道，会自动取反互补通道的PWM模式，因此有效电平都采取高电平

![image-20241025143505654](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025143505654.png)

死区时间计算，MOS管的死区时间合理为 （110-96）+（43-16） 乘上1.2 约50ns 需要50ns，

![image-20241025152156229](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025152156229.png)

一般都是0分频 也就是168Mhz

![image-20241025154820034](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025154820034.png)

![image-20241025154940265](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025154940265.png)

![image-20241025155023032](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241025155023032.png)

 



T DTS 也就是时间系数 2次方CKD[1:0] 一般都是0，也就是1/168Mhz=5.7ns

死区时间填0xFF 高三位111那么选第四条公式（32+31）* 16 * 5.7 = 5.8us

实测波形5.88us左右 因此推算成立 