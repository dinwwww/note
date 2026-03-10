# Modbus 主从协议栈解析 以及移植free Modbus

1、主从方式，只有主机发出消息，丛机之间不能互相通信，主机在同一时刻只能发起一个Modbus通信

2、一对一

3、广播模式

Modbus RTU

![image-20250217164449438](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250217164449438.png)

每个帧结束之间，至少3.5个字符

![image-20250217164729396](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250217164729396.png)

从机异常反馈

![image-20250217170439762](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250217170439762.png)

寄存器功能分为2类，离散输入和输入寄存器为只读功能，线圈和保持寄存器为读写功能

解析读寄存器源码

![image-20250225173754222](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225173754222.png)

在函数调用的时候，usAddress被自动＋1，因此要减回去

![image-20250225173855361](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225173855361.png)

获取到的值

自动++解析地址

![image-20250225173957662](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225173957662.png)

首先判断从机地址是否满足宏定义并且要在一定范围内

![image-20250225174130020](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225174130020.png)

判断你要访问的地址与启示地址的差值，例如1003-1000

![image-20250225174254679](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225174254679.png)

这是代码中的关键部分，指针操作：

- `pucRegBuffer++` 是 **指针加加操作**，即在每次将数据写入缓冲区之后，指针会自动增加，使得下次写入数据时，会写入到缓冲区的下一个位置。
- 这样，每次将寄存器的高字节和低字节分别写入缓冲区，从而将每个寄存器的 16 位数据拆分成两个 8 位的数据（Modbus 协议使用的是 16 位数据格式）。

![image-20250225175334500](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225175334500.png)

## 保持寄存器源码解析

相比于其他多了个模式的判断

![image-20250226092826342](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250226092826342.png)

判断读写，再在这个数组里面进行操作，先传高八位

![image-20250226093303320](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250226093303320.png)

![image-20250226093849648](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250226093849648.png)

离散输入的实现，离散输入是参考输入寄存器定义及其宏定义，只是离散输入单位是bit单独只进行位操作

![image-20250226101236509](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250226101236509.png)

这里的的下标是固定的寄存器位置，不是数组

![image-20250226102448573](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250226102448573.png)

iRegIndex 是输入的地址减去起始地址，那么就有下标，然后判断是在哪一个数组里面，就做三目运算判断，假设访问第八位，那么就是16/9=0 访问的第一个数组，假设地址大于16那么就能算出是在哪个数组里面的下标

先判断访问的i当前数组下标index是否满足1，如果满足1，用新的变量disinputBuf接受那一位，用或运算

![image-20250226103903548](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250226103903548.png)

当填充满8位，就要把值传出去，并且指针自增

![image-20250226104045515](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250226104045515.png)

![image-20250226104125502](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250226104125502.png)

## 保持寄存器的实现

和离散输入是一样的功能，只是多了个读写功能码，因此只看后面怎么操作写，进来先判断第一个位是不是1，如果是1那就直接写1，如果不是1，那么取反取0。他是在每次读写的时候，先判读读到是不是1，如果是1就把寄存器写1，不是1就写0，然后就能按顺序排序下去，coilVal只是临时存储读到的数据