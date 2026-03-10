# HAL库启动systick流程

首先要启动这个函数，这样就会初始化

![image-20250214135535027](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250214135535027.png)

这里做了hal的systick初始化流程

![image-20250214135620870](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250214135620870.png)

看到这个函数，不管怎么样他会先获取当前设定时钟频率，再分频计算出1ms的systick数值是多少，写入寄存器

![image-20250214135651432](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250214135651432.png)

但是默认的

![image-20250214135910669](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250214135910669.png)

这个会有问题，因为不是获取的最新值，所以其实这句话没什么太大用处，关键看这个RCC时钟配置，在RCC时钟配置之后，会修改出正常的主频频率

![image-20250214140109468](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250214140109468.png)

其中是这里获取主频频率，然后再分频计算出systick所需时间

![image-20250214140218725](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250214140218725.png)

因为systick是内核级外设，因此0-15可以选择，而且systick不分频系统主频，直接就是168Mhz，所以配置完成，每1ms进入中断，完成这个haldelay的使用