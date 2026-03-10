# Ti SVPWM与常规SVPWM对比，且进行标幺化处理分析

在推导出那么X=√3Ts/Udc *[Uβ] 		Y=√3Ts/Udc[√3Uα/2+1Uβ/2]		Z= √3Ts/Udc[-√3Uα/2+1Uβ/2]，此时并未进行标幺化处理，那么对整体除上Udc/√3，最后可以把Udc化简掉，得到2/√3,这个系数可以sin60 cos60系数化简，方便计算

此时对扇区进行分析

![image-20250114100054871](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114100054871.png)

Uα=T1/T * U0+T3/T  * U60 * cos60° 

Uβ=T3/T * U60 * sin60° 

化简切，U60=2/3Udc代入，那么可以获得标幺系统，且Udc化简掉

![image-20250114100326497](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114100326497.png)

此时在把T向左除去，那么T也可以变成标幺值

![image-20250114100346531](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114100346531.png)

同理扇区23456分析也可以依据这样

扇区2

![image-20250114100422026](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114100422026.png)

扇区3

![image-20250114100435168](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114100435168.png)

扇区4

![image-20250114100448625](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114100448625.png)

扇区5

![image-20250114100511951](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114100511951.png)

扇区6

![image-20250114100551819](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114100551819.png)

排列的具体顺序是：每个参考矢量分解后以零矢量 O000 (000)开始作用，接着是 开关状态只含有一个“1”的矢量作用，紧跟着作用的矢量其开关状态含有两个 “1”，最后作用的为另外一个零矢量 O111 (111)，这里是上半个PWM周期的基本 矢量作用顺序。 因为采用对称的七段调制，那么下半个PWM周期则以零矢量 O111 (111)开始作 用，接着是开关状态含有两个“1”的矢量作用，紧跟着作用的矢量其开关状态 只含有一个“1”，最后作用的为零矢量 O0 (000)。

那么开关顺序可以拆分出来，显而易见，开关状态只含有一个“1”的矢量有 U0 、U120 、U240 。开关状态含有两个“1”的矢量有 U60 、U180 、U300 。

![image-20250114101312782](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114101312782.png)

![image-20250114101330147](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114101330147.png)

此时判断扇区和常规SVPWM不同，其实就是线性规划的运用，根据三条直线大于零和小于零的组合确定扇区,**一般地，求线性目标函数在线性约束条件下的最大值或最小值的问题，统称为线性规划.线性约束条件的解叫做可行解，由所有可行解组成的集合叫做可行域。决策变量、约束条件、目标函数是线性规划的三要素.**

![image-20250114102001486](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114102001486.png)

本质上可以化简出来三条线，分别Y=0，Y=-√3 ,Y=√3，这三条线划分区间，刚好对应6个扇区划分

![image-20250114103159013](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114103159013.png)

​								![image-20250114103213442](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114103213442.png)

​								分析当temp2＞0时，那么此时就选中12扇区，y＞-√3这条曲线，然后temp3＞时，y＜√3这条曲线，那么此时扇区要-1也就是变成1，然后再分析temp1，是否对调扇区，就可以判断出来扇区位置

![image-20250114113133132](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114113133132.png)

根据开关顺序，可以编排如下开关矢量，先是000 然后 010 011 111 减少mos管的损耗，，又因为是中央对称模式，因此一个循环之后再到第二个只过了一半的pwm周期，那么有如下图

![image-20250114143138623](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114143138623.png)

![image-20250114143255663](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114143255663.png)

因此计算1号扇区的时间，T0/4=TPWM-T1-T2/4=TPWM /2* (1- T1 * -T2*)/2 ,此处标幺化处理，将Tpwm提出来 

![image-20250114143313333](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114143313333.png)

然后再到010这种有一个开关mos的矢量，这个时刻，那么就是同样转换成标幺化，把Tpwm提出来

![image-20250114143508876](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114143508876.png)

![image-20250114143712047](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114143712047.png)

明显可以得到 aon t < bon t < con t 这样的关系。这里的abc字母不代表电机三相序号， 而是代表导通的优先顺序，a代表最先导通，b代表接着导通，c代表最后导通。 以上求出了切换时刻对应的占空比，再次整理如下，可以看出，时间都是一样的计算，只是合成出来不同方向的矢量，时间相同，但是导通的管子取决于扇区

![image-20250114143751246](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114143751246.png)

![image-20250114143920430](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114143920430.png)

由最开始推导的T*这个参数的标幺化，那么可以推导出，T3指的是011二进制判断，T1就是001只有一项导通，

![image-20250114144221302](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114144221302.png)

![image-20250114144340889](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114144340889.png)

为了减少mos损耗，那么按照规律可以得出，先作用与后作用

![image-20250114144426932](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114144426932.png)

根据前面推导的切换时刻表，可以推导出来，把上文t1 t2带入化简

![image-20250114144700287](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114144700287.png)

又因为，时间虽然是一样，但是哪几个矢量不一样，必须满足先0 100 110 111这种方向，taon 和 tbon 和 tcon都是从低到高这样编排，值虽然一样，但是作用的矢量不一样，因此可以区分出来例如在U60-U120这个扇区，可以看到，必须先000再到010，再到011，这样合成出来复合顺序因此，Tb的时间选择是taon

![image-20250114145711581](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114145711581.png)

因此带入整理数据，可以发现虽然此时14扇区，先后作用taon tbon tcon 不一样，但是计算出来ABC三相占空比的值是一样的，同理25 46也是一样的             

![image-20250114145937297](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114145937297.png)

接着分析代码

![image-20250114151527317](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114151527317.png)

那么此时用temp来代表扇区划分，且带入，因为此时扇区14 25 46 α与β都不一样，虽然他们计算公式是一样，但是代表的值大小是不一样，所以可以推出这个算法成立，并且在计算的时候，前面的1可以提出来，那么就是一半的占空比，对应上代码就是

![image-20250114151609663](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114151609663.png)

![image-20250114151916240](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114151916240.png)

在代码的时候，ti是设计递减技术模式

![image-20250114152028425](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114152028425.png)

但我们实际工程用的时候，不是递减计数，是递增计数，因此不用1-Ta这样计算，中央对齐模式1递增计数，PWM模式2，大于该值时拉高，因此推断成立

![image-20250114152155250](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114152155250.png)

![image-20250114152254931](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250114152254931.png)