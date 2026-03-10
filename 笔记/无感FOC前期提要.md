# 无感FOC前期提要

## 永磁同步电机的分类

- 隐极式和凸极式电机

对于同步发机（包括永磁同步发机）从转子的外表面分为隐极式和凸极式电机，转子大多数为圆饼状（直径大），转子上有明显凸起的成对磁极（和励磁线圈），易安装多对磁极，一般都是用于[水轮发电机](https://zhida.zhihu.com/search?content_id=154689850&content_type=Article&match_order=1&q=水轮发电机&zhida_source=entity)。

而隐极式发电机，转子呈圆柱形，在圆柱表面开槽以安装励磁绕组或者永磁体，电机具有均匀的气隙。工艺较复杂，适用于极数较少，转速快的[汽轮发电机](https://zhida.zhihu.com/search?content_id=154689850&content_type=Article&match_order=1&q=汽轮发电机&zhida_source=entity)。

**对于永磁同步电机，凸极还是隐级主要是看交直轴的磁路**，比如[表贴式永磁电机](https://zhida.zhihu.com/search?content_id=154689850&content_type=Article&match_order=1&q=表贴式永磁电机&zhida_source=entity)，由于永磁体贴在转子铁芯表面，而且永磁体的相对磁导率几乎为1，所以交直轴磁路磁阻是相同的，即为隐级式。而凸极式结构，永磁体的安装方式造成了交直轴磁阻不等而形成的。最本质的区别在于两者交直轴电感是否相等，隐极性电机交轴电感 Lmq 和 直轴电感 Lmd相等，而凸极性电机Lmq不等于Lmd。

![image-20250225144232290](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225144232290.png)

**转子磁极轴线称为直轴**，并且主磁通必定经过永磁体，对于**磁通密度较高的永磁体其磁导率几乎与空气磁导率相同，这意味着永磁体可近似认为是空气，相当于增大了气隙**。当直轴或永磁体轴线与钉子轴线重合时，**定子的电感即为直轴电感**。再将永磁体转过90°电角度，此时定子磁场与转子极间轴线正对，即磁路中不存在永磁体。在此位置下测得的电感称为交轴电感，电感的大小与电机尺寸、气隙长度和绕组匝数有关 .

下面是各种磁体安装方式下的交直轴位置，在此位置测得的定子的电感称为**定子的交轴或者直轴电感**。

![image-20250225144352407](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225144352407.png)

## 永磁同步电机电感数学模型推导

PMSM的数学模型中，最难理解的莫过于各种电感，包括自感 LA 、互感LAB、[漏感](https://zhida.zhihu.com/search?content_id=249129004&content_type=Article&match_order=1&q=漏感&zhida_source=entity)Lsσ、[励磁电感](https://zhida.zhihu.com/search?content_id=249129004&content_type=Article&match_order=1&q=励磁电感&zhida_source=entity)Lm1、[等效励磁电感](https://zhida.zhihu.com/search?content_id=249129004&content_type=Article&match_order=1&q=等效励磁电感&zhida_source=entity)Lm、[同步电感](https://zhida.zhihu.com/search?content_id=249129004&content_type=Article&match_order=1&q=同步电感&zhida_source=entity)Ls、[直轴同步电感](https://zhida.zhihu.com/search?content_id=249129004&content_type=Article&match_order=1&q=直轴同步电感&zhida_source=entity)Ld、[交轴同步电感](https://zhida.zhihu.com/search?content_id=249129004&content_type=Article&match_order=1&q=交轴同步电感&zhida_source=entity)Lq。这么多的概念特别容易混淆，但这些概念在电机控制中又显得非常重要，因为电感是衡量电流产生磁链的能力的物理量，而磁链又直接关系到电磁转矩的大小，因此，在我们推导永磁同步电机数学模型之前，有必要先来学习一下永磁同步电机中的电感。

![image-20250225144457660](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225144457660.png)

也就是对应A处的磁链简化成

![image-20250225144518889](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225144518889.png)

但是还要考虑互感磁链

![image-20250225144533077](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225144533077.png)

三相电流每时每刻的互感相同

![image-20250225144554520](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225144554520.png)

并且自感相同

![image-20250225144622530](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225144622530.png)

三相电流合为0

![image-20250225144642361](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225144642361.png)

![image-20250225144714446](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225144714446.png)

因此正常情况下，自感和互感是不会改变的，是个固定的值

![image-20250225145044386](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225145044386.png)

三相电机，磁链方程的推导，在PMSM中只有两个来源，一个是定子绕组通电的磁链，上文推导过，另一个是永磁的本身固有的磁场，所以合成的磁链就是φf来表示

![image-20250225152720676](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225152720676.png)

因为PMSM的电感矩阵是不会改变，但是IPMSM的自感和互感会随着转子位置的改变而改变，这种改变类似二次谐波，因此要带入cos2θ来表示这种谐波

那么可以化简

![image-20250225152837840](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225152837840.png)

![image-20250225153003536](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225153003536.png)

这个矩阵看起来非常复杂，但是仔细分析一下就会发现，也没那么复杂。对角线上的元素表示电机的自感，由两部分组成，直流分量和与转子位子呈2倍关系的正弦分量；非对角线上的元素表示相间的互感，也由两部分组成直流分量和与转子位子呈2倍关系的正弦分量

![image-20250225153203624](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225153203624.png)

最后化简整个矩阵

![image-20250225154935162](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225154935162.png)

Ld和Lq的表达式并不需要知道当前定子的电流大小，参考上文Ld Lq的推导，因此

![image-20250225155524964](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225155524964.png)

最后

![image-20250225155629237](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225155629237.png)

αβ坐标系下的数学模型是直接通过abc坐标系下的数学模型变换来的，其中的理论并不复杂，只有一些三角恒等式和变量替换，但是计算过程比较长而已。因此，我们将在这一小节中给出详细推导过程，供读者参考。

利用[Clark变换](https://zhida.zhihu.com/search?content_id=249747742&content_type=Article&match_order=1&q=Clark变换&zhida_source=entity)可以将三相坐标系变换到两相坐标系下，但是该变换存在两种类型，等幅值变换和等功率变换，在[如何用空间矢量法实现SVPWM？](https://zhuanlan.zhihu.com/p/786046983)一文中，我们已经讲清楚了二者的区别，这里我们以等幅值变换为例来进行分析。等幅值Clark变换矩阵如下

![image-20250225155818968](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225155818968.png)

![image-20250225155832876](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225155832876.png)

![image-20250225155857779](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225155857779.png)

转换成电压方程

![image-20250225160007670](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225160007670.png)

![image-20250225160111432](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225160111432.png)

dq轴同理，用的帕克变换带入，具体推导看知乎

![image-20250225160139461](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250225160139461.png)

## 无感状态观测器控制器引入

实际电路上的状态

Ut 实际上是Uα Uβ

Xt是Iα Iβ θ W

![image-20250227142028798](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250227142028798.png)

虚拟的观测系统 假设实际的Iα Iβ 与 估算的Iα Iβ 误差很小，那就可以认为，估算出来的值和实际的值是一样的，也就是估算出来

Iα Iβ θ W 

**![image-20250227142249790](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250227142249790.png)**

状态观测器是多输入多输出的控制方式 

![image-20250227153505581](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250227153505581.png)

简化同步方程与状态空间方程表达式做转换

![image-20250227154153594](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250227154153594.png)

化简变量

![image-20250227154539766](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250227154539766.png)

用方程组去转换矩阵形式

化简出来矩阵

![image-20250227155913321](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250227155913321.png)

X为

![image-20250227155925764](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250227155925764.png)

Fx的形式

![image-20250227160000526](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250227160000526.png)

B矩阵

![image-20250227160028928](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250227160028928.png)

C矩阵

![image-20250227160051835](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250227160051835.png)

最后把矩阵方程组转换为几个大的矩阵相乘

![image-20250227160130899](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250227160130899.png)

## 卡尔曼滤波器可以融合多个滤波器，统称数据融合算法，就是不同传感器有不同的比重，卡尔曼融合之后，误差会比任何一个传感器都小

![image-20250228103541173](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228103541173.png)

1、用模型去模拟一个传感器

2、用温度传感器测量温度传感器的值

3、确定权重

卡尔曼滤波器的过程

1、预测方程 X是估计值，从K-1步到K步对X的估计值 φ是指系统构建模型

2、计算预测均方差，P预测均方差，Q是指预测过程中的误差（平方），预测的误差固定值

3、计算卡尔曼增益K R实际传感器的误差（平方）

4、修正估计值 Y就是实际模型和预测模型的差。但是预测模型要乘C，反馈回去就是要乘K增益，因此如果K取值很小，那么就完全没用到另外一个传感器的值是用纯模型值，但是用了K那么传感器占比就会增大 

5、用于下一个时刻的均方差，PK I矩阵是单位矩阵 

![image-20250228111520835](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228111520835.png)

![image-20250228111206353](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228111206353.png)

实际推导过程

假定条件

![image-20250228113328548](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228113328548.png)

假定时间间隔很小，那么系统模型就是1，因为温度不会突变

因此K1 =0.4

估计出T1时刻的值

![image-20250228113608214](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228113608214.png)

本质就是动态调节K值，改变占比，那么就会逐渐逼近

![image-20250228114136633](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228114136633.png)

补充五个方程前提

E是期望值 两个期望值相称为0，那么表示这两个不相关

Wk和Vk都是噪声

![image-20250228141910361](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228141910361.png)

Q是代表方差，也就是误差的平方，带入矩阵就是本身乘T

![image-20250228142315736](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228142315736.png)

卡尔曼滤波是一种递推形式的滤波，是线性的系统，最小方差估计 

重要概念，估计误差

![image-20250228142739178](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228142739178.png)

## 推导方程1 X是指预测的值，由条件均值方程，Xk 和Y1|k-1，意思是由模型观测出来的值和实际的值做条件均值

![image-20250228170124404](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228170124404.png)

![image-20250228170556635](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228170556635.png)

系统矩阵可以提出 那么推导出来

![image-20250228170640552](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228170640552.png)

又因为估计值的公式

![image-20250228170740057](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228170740057.png)

 那么推导最后

![image-20250228170838775](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250228170838775.png)

## 预测均方差推导，因为Pk k-1是个均方差，因此要用矩阵的数学期望

![image-20250303150443211](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303150443211.png)

预测值是 =真实值-预测值 

![image-20250303150559742](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303150559742.png)

把真实值Xk 和预测值Xk|k-1替换化简

![image-20250303151053326](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303151053326.png)

把刚刚的值带入转秩矩阵

![image-20250303151522561](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303151522561.png)

因为有两个不相关量 期望为0，可以化简2个，因此只有第一项，第四项，所以可以形成递推关系，最后关系如下，第四项是误差的矩阵Q

![image-20250303151854174](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303151854174.png)

## 修正值估计

预测值＋上修正值 关键在于取K 也就是卡尔曼增益，估计值是和真实的误差的方差是最小的，所以就是最小方差估计

![image-20250303160452742](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303160452742.png)

本次求的是K时刻的误差，也就是用K时刻真实值-减去K时刻的估计值

![image-20250303161921821](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303161921821.png)

提取出来，化成单位阵

![image-20250303162247252](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303162247252.png)

因为是要求方差最小，也就是K时刻的估计值方差最小，同理求期望的话，还是求转秩

![image-20250303162551311](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303162551311.png)

化简结果

![image-20250303163049892](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303163049892.png)

要把这个期望调到最小，只有K是变量，其他都是定值，可以使用求导极值为0 的点，因为这个是个实对称矩阵，要使得斜方差最小那么就要主对角线上和最小，也就是迹最小 tr

![image-20250303164500970](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303164500970.png)

化简P是实对称P=Pt

![image-20250303164643788](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303164643788.png)

对迹求偏导常识

![image-20250303164827338](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303164827338.png)

对迹求偏导

![image-20250303165728605](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303165728605.png)

令偏导为0，那么有可能K值就是极值点把K 移到一遍去

![image-20250303170215338](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303170215338.png)

其他公式大差不大暂且不推论

## 控制永磁同步电机，非线性系统线性化，连续系统离散化才能导入单片机使用

线性化的本质，就是泰勒展开

![image-20250303173852964](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303173852964.png)

电机实际的非线性系统x是电机当前状态，u是输入，t是时间 w是扰动

![image-20250303174125961](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303174125961.png)

展开1阶无穷小，然后化简替换可以得到约等于

![image-20250303174656847](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303174656847.png)

连续系统变成离散化，其实就是求导，这个Ts就是△x

![image-20250303175125865](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303175125865.png)

对这个公式进行离散化

![image-20250303180559475](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303180559475.png)

![image-20250303180727164](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303180727164.png)

![image-20250303180739742](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250303180739742.png)