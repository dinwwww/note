# STM32 can总线 Controller Area Network Bus

 ![image-20250207140028910](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250207140028910.png)

全双工和半双工的区别

![image-20250207140914374](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250207140914374.png)

单端信号和差分信号的区别就是一个有共地差，一个没有

can总线最大的优点就是可以多个主控一起通讯

![image-20250207141805956](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250207141805956.png)

## CAN总线硬件说明

一般使用闭环CAN总线，单片机输出TTL电路电平，要经过电平转换芯片，转换成差分信号，终端电阻加入120Ω电阻，且阻抗匹配，防止终端回波现象，会使信号终端跳变沿变得平稳。在没有信号传输的时候，终端电阻可以使两端线电压拉到同一水平，总线默认状态就是1

![image-20250207143144353](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250207143144353.png)

开环CAN不接受回波，终端电阻

![image-20250207143842067](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250207143842067.png)

在CAN 逻辑表示1，其实是隐形电平，但实际电路上显示的收紧状态也就是两根线电位一样，电位差为0。逻辑表示0，实际电路是放开状态，电位不一致有差值，所以表示显性，当电路中同时出现显性电平和隐性电平，就会出现0大于1的效果，显性电路干扰隐性电路

![image-20250207144733885](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250207144733885.png)

高速CAN与低速CAN使用收发器不一样

## 数据帧解析

数据帧属于广播式发送数据，ACK是应答位，发送放发送隐性1，接受位发送显性0

![image-20250207161638917](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250207161638917.png)

先是帧起始，在帧起始之后，有报文ID，其实报文ID+RTR帧可以构成仲裁段，区分RTR 0/1那就可以读或者写，与ID结合就构成仲裁段，IDE是扩展格式标志位，区分是扩展格式还是标准格式，r0属于保留位，DLC是表示数据段的长度，表示后面数据段长度是多少1-8字节，在ACK段先释放总线，然后如果接收方拉紧总线，ACK界定槽然后再去读取这个总线，如果不是释放状态，说明其他接收到了，也就表示发送成功，如果还是释放状态，就可以知道发送状态是怎么样的。ACK槽可以允许多个总线去拉开，这样可以同时被多个设备接受。

![image-20250207164606534](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250207164606534.png)

遥控帧本意就是要一个设备发出一个信号，让别的设备发出指定需要的数据，等于两个数据帧，一个发送另外一个也发送。所以遥控帧没有数据段，RTR改为隐性1

位填充规则，在can总线上波形会有区别，但是实际寄存器接受都是一样的

![image-20250207170206541](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250207170206541.png)

在CAN总线接收机制分析

![image-20250208091713783](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208091713783.png)

主要会因为时钟问题导致不同步

![image-20250208092028686](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208092028686.png)

位时序机制在每个位都有四个段，每个段都有最小时间单位组成Tq，

SS表示同步段，这个段要和跳变沿重合最好

PTS传播时间段，这个段是消除物理网络上的延迟

PBS1 PBS2 相位缓冲段，确定采样点位置采样点在PBS1 PBS2之间，从而确定采样点位置

![image-20250208094104720](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208094104720.png)

硬件同步机制，由于是异步通讯，每个设备都有自己的时钟周期，当发送方开始一个发送位SOF时候，所有的接受方收到SOF的下降沿的时候，会把自己的时序拨到ss位，这样就能保持同步状态

![image-20250208094942390](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208094942390.png)

再同步的设计是自动补偿，SJW最大补偿宽度

![image-20250208095304314](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208095304314.png)

计算波特率

![image-20250208095405955](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208095405955.png)

总线仲裁机制，在有设备发送数据的时候，其他数据要发送，那么就会等到这个发送结束检测到空闲状态再尝试发送

![image-20250208100719600](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208100719600.png)

仲裁机制2，在多设备的发送的时候，看ID号，并且有线与特性，且回读机制

![image-20250208110829698](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208110829698.png)

仲裁过程，当两个设备同时发送的时候，每个设备发出01相同，到ID号不同的时候，单元1发送1，单元2发送0，此时总线由于线于特性，总线保持就是显性0的状态，单元1回读回去发现不对，单元2回读回去发现对，那么此时仲裁就结束了，因为单元2回读一致，单元1就会暂停发送，等待下一次的发送，因此ID号越小，发送的优先级越高

![image-20250208111000520](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208111000520.png)

数据帧和遥控帧的仲裁时候，多了一个位，因此数据帧大于遥控帧优先级



CAN的错误处理机制，每个设备都有权利发出错误标志，破坏这个数据，当发出错误标志整个总线都会停止接受这个数据，那么此时如果某个设备一直发出错误数据，也就是会发生总线一直处于错误状态

![image-20250208111700129](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208111700129.png)

回顾错误帧，有两种错误标志位 

主动错误标志位六个显性0，会破坏总线

被动错误标志六个隐性1 不会破坏总线，那么就会避开这个设备

每个设备一开始都是初始状态，主动错误状态太多，进入被动错误状态，再太多就会进入总线关闭状态

TEC REC就会显示当前错误的频率

![image-20250208141527582](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208141527582.png)

![image-20250208141827098](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250208141827098.png)

STM32中CAN外设的基础逻辑

带有一个接受发送fifo，还有一个过滤器可以过滤掉不用的报文

![image-20250210092259456](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250210092259456.png)

报文邮箱具有自动重发流程

![image-20250210093120623](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250210093120623.png)

CAN外设比较重要就是过滤标识符

可以选择列表模式，也可以选择过滤模式，可以选32位或者16位

在过滤模式的时候，R1寄存器先写ID号，前三位，然后再屏蔽寄存器R2写111就可以选择R1寄存器里面前三位对应的就满足，其他全过滤掉，类似IP地址掩码的作用

![image-20250210104852948](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250210104852948.png)

在填写过滤数据的时候，存储数值是小端对齐右对齐，所以需要左移5位，才能满足R1寄存器，在与其他值或

![image-20250210110022458](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250210110022458.png)

![image-20250210111359814](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250210111359814.png)

计算位时序，和基础CAN协议有点不同，大体是一致的，只是做了融合

![image-20250210112151666](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250210112151666.png)

CAN外设也拥有独立的中断

![image-20250210112313079](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250210112313079.png)

同步时间戳，可以详细记录SOF时刻什么时候发送报文与接受报文

![image-20250210112613310](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250210112613310.png)