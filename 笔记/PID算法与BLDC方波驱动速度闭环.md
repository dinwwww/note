# PID算法与BLDC方波驱动速度闭环

## PID算法

PID是;Proportional比例，Integral积分，Differential微分。首字母缩写组成

### 比例环节

比例环节可以成比例地反应控制系统的偏差信号，输出与输入偏差成正比，用来减少系统偏差。

![image-20241010140456096](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241010140456096.png)

![image-20241010140504705](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241010140504705.png)

当e确定时，Kp越大输出u越大，那么调节力度越大。当Kp值越大，达到目标的时间就越短，与此同时，会出现一定幅度的超调与振荡，这会使得系统的稳定性下降。同时如果只有Kp调节的话，会带来另外一个影响，那么就是稳态误差。

如何来的呢，当Kp的提升能力有限，又被外部系统所影响的时候就会造成稳态误差。如果此时把Kp动态的调节放大来消除误差，但是我们无法确定这个外部影响的因素是个定值，就有可能调节造成超调和振荡。因此需要引入积分环节。

## 积分环节

积分环节可以对偏差e进行积分，只要存在偏差，积分环节就会不断的起作用，主要用于消除静态误差，提高系统无差度

![image-20241010142155654](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241010142155654.png)

假设当前比例环节被外部影响抵消，此时存在偏差，积分环节一直累加，以此增大输出，从而消除静态误差，当系数Ki输出越大时，输出就越大，系统消除静态误差的时间就越短。但是此时Ki越大，调节达到目标的时间越短，但是会出现一定幅度的振荡和超调，如果系统一直存在偏差，积分环节会不断累计偏差，当偏差为0的时候，没有偏差，但是积分环节任然存在，十分容易发生超调现象。因此需要引入微分环节，提前减弱输出，抑制超调现象。

## 微分环节

微分环节反应偏差量的变化趋势，根据偏差量的变化趋势，提前做出响应控制，减少超调，抑制振荡。微分环节比较的是偏差的偏差，如果变化趋势过快，例如当前偏差是5，上一次偏差是10那么偏差的偏差就是负数，就会抑制输出，削弱比例积分控制器的环节。

![image-20241010145221411](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241010145221411.png)

## PID算法离散公式

### 位置式PID

![image-20241010150002360](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241010150002360.png)

### 增量式PID

### ![image-20241010152045221](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241010152045221.png)

### 调节PID选择

![image-20241010152456920](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241010152456920.png)

### 采样周期的选择

采样定理，又称香农采样定律、奈奎斯特采样定律，是[信息论](https://baike.baidu.com/item/信息论/302185?fromModule=lemma_inlink)，特别是通讯与[信号处理](https://baike.baidu.com/item/信号处理/84717?fromModule=lemma_inlink)学科中的一个重要基本结论.E. T. Whittaker（1915年发表的统计理论），[克劳德·香农](https://baike.baidu.com/item/克劳德·香农/565097?fromModule=lemma_inlink) 与[Harry Nyquist](https://baike.baidu.com/item/Harry Nyquist/2560569?fromModule=lemma_inlink)都对它作出了重要贡献。另外，V. A. Kotelnikov 也对这个定理做了重要贡献。

采样是将一个信号（即时间或空间上的[连续函数](https://baike.baidu.com/item/连续函数/2716812?fromModule=lemma_inlink)）转换成一个数值序列（即时间或空间上的[离散函数](https://baike.baidu.com/item/离散函数/10626142?fromModule=lemma_inlink)）。

采样得到的[离散信号](https://baike.baidu.com/item/离散信号/6613954?fromModule=lemma_inlink)经保持器后，得到的是阶梯信号，即具有[零阶保持器](https://baike.baidu.com/item/零阶保持器/1153199?fromModule=lemma_inlink)的特性。

如果信号是带限的，并且[采样频率](https://baike.baidu.com/item/采样频率/1494233?fromModule=lemma_inlink)高于信号最高频率的一倍，那么，原来的连续信号可以从采样样本中完全重建出来。

带限[信号变换](https://baike.baidu.com/item/信号变换/5921898?fromModule=lemma_inlink)的快慢受到它的最高频率分量的限制，也就是说它的离散时刻采样表现信号细节的能力是非常有限的。采样定理是指，如果[信号带宽](https://baike.baidu.com/item/信号带宽/10425055?fromModule=lemma_inlink)小于[奈奎斯特](https://baike.baidu.com/item/奈奎斯特/9893523?fromModule=lemma_inlink)频率（即采样频率的二分之一），那么此时这些离散的[采样点](https://baike.baidu.com/item/采样点/4779416?fromModule=lemma_inlink)能够完全表示原信号。高于或处于[奈奎斯特频率](https://baike.baidu.com/item/奈奎斯特频率/6047805?fromModule=lemma_inlink)的频率分量会导致**[混叠现象](https://baike.baidu.com/item/混叠现象/24417927?fromModule=lemma_inlink)**。大多数应用都要求避免混叠，混叠问题的严重程度与这些[混叠频率分量](https://baike.baidu.com/item/混叠频率分量/2572628?fromModule=lemma_inlink)的[相对强度](https://baike.baidu.com/item/相对强度/12729990?fromModule=lemma_inlink)有关

![image-20241010153140333](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241010153140333.png)

也就是采样频率要高于被控物体最高频率的一倍。

![image-20241010153256059](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241010153256059.png)

## 霍尔计算转速

![image-20241010170111569](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241010170111569.png)