# SVPWM

## 引入

FOC控制电机旋转，就要产生旋转磁场，要使得磁场旋转就需要相位相差120度的正弦电流，要产生正弦电流就需要产生正弦电压，在正弦电压如何产生，利用平均等效法，也就是改变PMW波的占空比，产生大小不同电压（spwm）

通过前面六步换相可以知道，在BLDC中，可以导通六步换向中，任意相邻的两个合成方向矢量，再加上0矢量，就可以完成在这相邻的矢量任意角度合成旋转

在合成的过程中，相邻两个向量比例不变，因为比例是决定合成矢量角度，但是增加相邻矢量长度，同样的可以增加合成矢量强度，从而力矩变大

在SVPWM模块中，输入的是Uα，Vβ，输出的是三相计数器比较值

在FOC中，同步指的是磁场与电机磁极同步旋转，并不是指的磁场与电机磁极同步，其中存在角度差，这样才会有力矩进而旋转，那么90°是力矩最大时刻，如果超过90°就会失步

## 克拉克变化

从Ua Ub Uc→Uα Uβ

![image-20241016100625000](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016100625000.png)

做向量的合成

Uα = Ua-1/2Ub-1/2Uc 

Uβ= 0 + √3/2Ub-√3/2Uc

其中在克拉克变换中，矢量合成之后，三相电压合成后幅值会变成原来3/2倍，因此需要在向量的合成这块，多乘上2/3维持幅值大小

图下推导使用到欧拉公式的变形形式，最后带入计算可以得出Vs值 ，过程比较繁杂，基本计算过程在下面图示



![image-20241016101323174](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016101323174.png)

![image-20241016111759997](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016111759997.png)

![image-20241016113842432](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016113842432.png)

![image-20241016113937693](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016113937693.png)

![image-20241016114026769](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016114026769.png)

零点饱和位，意思就是三相四线制，有一个零点位u0，就是有流入与流出，那么可以推出最后的克拉克公式

![image-20241016115800138](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016115800138.png)

![image-20241016115733737](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016115733737.png)



## 判断扇区

等幅值变换的话要带入2/3这个值，也就是从三个方向矢量变成两个方向矢量，在欧拉公式且图示下，这两个矢量相差90度，那么可以得到Uα=Umsinθ，Uβ=Umcosθ，带入1图求解也可以化简得到

Uα = Ua-1/2Ub-1/2Uc 

Uβ= 0 + √3/2Ub-√3/2Uc

这里面的θ不是电角度，而是三相正弦波的相位

最后简化出来是二维坐标系，那么Uα=Umcosθ，Uβ=Umsinθ，进行扇区判断的话，就是要根据θ角度，也就arctan（Uα/Uβ）就得出扇区判断角度。

### 一般不用arctan来求解扇区，在单片机求解计算量比较大

### 提前区分扇区

1	tan0<θ<tan60					0<θ<√3					Uβ>0			Uα>0					推出 Uβ/Uα<√3	

  凑成反克拉克形式 			√3Uα/2-Uβ/2>0								-√3Uα/2-Uβ/2<0			

2	tan60<θ<tan120				√3<θ<-√3				Uβ>0			Uα>0,Uα<0		推出 Uβ/Uα<-√3   Uβ/Uα>√3  		

​								取Uα＞0时√3Uα/2-Uβ/2<0			取Uα<0时-√3Uα/2-Uβ/2<0

3	tan120<θ<tan180				-√3<θ<0			 	Uβ>0			Uα<0				推出 	Uβ/Uα>-√3							

​									√3Uα/2-Uβ/2<0											-√3Uα/2-Uβ/2>0

4	tan180<θ<tan240				0<θ<√3				  Uβ<0			Uα<0				推出 		Uβ/Uα<√3						

​									√3Uα/2-Uβ/2<0											-√3Uα/2-Uβ/2>0

5	tan240<θ<tan300				√3<θ<-√3			   Uβ<0			Uα>0,Uα<0		推出 Uβ/Uα<-√3   Uβ/Uα>√3  	

​									取Uα<0 推√3Uα/2-Uβ/2>0			取Uα>0 推-√3Uα/2-Uβ/2>0

6	tan300<θ<tan360				-√3<θ<0			     Uβ<0			Uα>0				推出 Uβ/Uα>-√3							

​													√3Uα/2-Uβ/2>0								-√3Uα/2-Uβ/2< 0

Uα=Umcosθ，Uβ=Umsinθ 蓝色β黄色α

![image-20241016140939048](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016140939048.png)

通过反克拉克变换，可以发现一定规律，分出六个扇区，有一个在一侧，另外两个必在另一侧，服务于扇区判断

![image-20241016143918065](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016143918065.png)

反克拉克变换，服务于扇区判断

![image-20241016144517985](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016144517985.png)

最后根据Uβ 与√3Uα/2-Uβ/2 								-√3Uα/2-Uβ/2			大小关系可以比较出来在哪个扇区

为什么选取Uβ不选择Uα，因Uβ是sin函数，在扇区判断中不会存在大于0与小于0同时出现的情况

黄色Uβ 蓝色√3Uα/2-Uβ/2  橙色-√3Uα/2-Uβ/2

那么SVPWM采用二进制 N=4C+2B+A 

A是Uβ B是√3Uα/2-Uβ/2 	C是-√3Uα/2-Uβ/2   只要大于0取1 得到就是扇区1-6

扇区1 2 3 4 5 6 

N	3 1 5 4 6 2

![image-20241016151324051](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016151324051.png)

### 相邻矢量作用时长

根据扇区计算 Sa Sb Sc分成三个部分扇区导通可以看成上半部分导通为1，那么此时下半桥截止,N为中性点，此时电压就有正有负，且可以通过电阻与电流分析电压大小 Udc是电源电压

Sa 		Sb 			Sc 		 VAN	VBN	VCN

1			0				0			2/3	-1/3	-1/3

0			1				0			-1/3	2/3	-1/3

1			1				0			1/3	1/3	-2/3

0			0				1			-1/3	-1/3	2/3

1			0				1			1/3	-2/3	1/3	

0			1				1			-2/3	1/3	1/3

因此六个方向上矢量大小为2/3Udc

### SVPWM电压利用率为什么提高到100

电压利用率 = 线电压幅值/母线电压

![image-20241016174308588](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016174308588.png)

![image-20241016174321632](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016174321632.png)

![image-20241016174333259](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016174333259.png)

由以上过程可以计算出Vout（欧拉公式逆用）

![image-20241016174419858](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016174419858.png)

Svpwm相电压幅值是2/3Udc

输出合成矢量电压（线电压），是相电压幅值的3/2倍，因此乘上就是Udc 再除去母线电压因此利用率是100

![image-20241016175243301](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016175243301.png)

## 为什么会生成马鞍波

分析中性点对地的电压Vng，不是固定的，得出是1/3 和2/3浮动 因此这个得出的电流是上下浮动的，因此线电压与相电压不是固定的根号3倍

![image-20241016180146931](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241016180146931.png)

### 存疑

马鞍波生成，母线利用率，线电压比母线电压，为什么。。。。等等

### 矢量作用时长关系

如何合成任意矢量

Ux与Uy作用时长不一样可以，合成这个扇区的任意矢量，本质就是矢量合成是由分向量长度变化

合成矢量方向不变，大小改变时候，减少Tn Tx Ty等比例加大

![image-20241017092251955](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017092251955.png)

### 作用时长Tx Ty Tn 关系计算 第一个扇区

输出Uout = Ux *Tx/Ts + Uy * Ty/Ts + Un * Tn/Ts

输入Uα Uβ

![image-20241017100454619](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017100454619.png)

其中

Uα=Umcosθ，Uβ=Umsinθ

![image-20241017101738718](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017101738718.png)

![image-20241017101814188](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017101814188.png)

![image-20241017101829484](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017101829484.png)

### 在第二个扇区角度要减去60度

![image-20241017102652663](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017102652663.png)

![image-20241017102659871](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017102659871.png)

化简结果

![image-20241017102807280](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017102807280.png)

另一矢量

![image-20241017103307858](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017103307858.png)

![image-20241017103328669](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017103328669.png)

整理成标准形式

![image-20241017103341618](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017103341618.png)

### 第三扇区 减去120度

![image-20241017104707113](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017104707113.png)

![image-20241017104725596](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017104725596.png)

化简

![image-20241017104805990](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017104805990.png)

对于Tx

![image-20241017105809984](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017105809984.png)

![image-20241017105841445](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017105841445.png)

最后的化简结果运用一些技巧，需要和扇区判断那几个值一样，减少代码负荷

![image-20241017105855013](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017105855013.png)

### 第四扇区 减去180度

![image-20241017110105445](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017110105445.png)

化简结果

![image-20241017110345208](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017110345208.png)

另一个方向

![image-20241017110909696](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017110909696.png)

直接化简可以得出

![image-20241017111135681](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017111135681.png)

### 第五个扇区 减去240度

![image-20241017113408290](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017113408290.png)

展开化简

![image-20241017113548698](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017113548698.png)

另一边向量

![image-20241017113839550](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017113839550.png)

化简

![image-20241017113901811](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017113901811.png)

最后的结果

![image-20241017113918134](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017113918134.png)

### 第六个扇区 减去300度

![image-20241017114019184](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017114019184.png)

![image-20241017114040061](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017114040061.png)

### 总结划分扇区

扇区								Tx 																			Ty

1									√3Ts/Udc[√3Uα/2-1Uβ/2]								√3Ts/Udc	*[Uβ]

2									√3Ts/Udc[1Uβ/2-√3Uα/2]								√3Ts/Udc[√3Uα/2+1Uβ/2]

3									√3Ts/Udc * 	[Uβ]											√3Ts/Udc[-1Uβ/2-√3Uα/2]

4									√3Ts/Udc * 	[-Uβ]											√3Ts/Udc[-√3Uα/2+1Uβ/2]

5									√3Ts/Udc[-√3Uα/2-1Uβ/2]								√3Ts/Udc[-1Uβ/2+√3Uα/2]

6									√3Ts/Udc[1Uβ/2+√3Uα/2]								√3Ts/Udc * 	[-Uβ]	



关联扇区判断	Uβ 与 	√3Uα/2-Uβ/2   与	 -√3Uα/2-Uβ/2

推导出来xyz相关和扇区判断有点区别，但是ti代码直接用XYZ来判断扇区

那么X=√3Ts/Udc *[Uβ] 		Y=√3Ts/Udc[√3Uα/2+1Uβ/2]		Z= √3Ts/Udc[-√3Uα/2+1Uβ/2]

​									Tx										Ty

1								-Z											X

2								 Z											Y

3								X											-Y	

4								-X											Z

5								-Y											-Z

6								Y											 -X

### 相邻时长转换成mos开关时长（定时器比较值）

通过扇区判断＋Tx Ty控制mos管来完成矢量的合成，UVW的三相Ta Tb Tc，使用中心对齐模式

其中和六步换相的扇区规划不同，六步换向实际上只有两相导通，另外一相始终关断，而FOC要求的是三相都开启，因此六步换向会与X轴有个30°角度差，而FOC这个扇区分布可以贴在X轴，且，为了避免mos管的损耗，每次改变只会改变一个mos的通断，因此这个分布可以看成6个合成出来矢量与2个空矢量

![image-20241017170046451](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017170046451.png)

### 在第一扇区

000变成100变成110变成111变成110变成100变成000，这样构成循环

那么扇区变化就是000 100 100 111计数，然后中央对齐模式会自动完成下半部分

计算出来Tn Tx Ty的时候，赋值只用一半，因为是中央对齐的，Sa三个都加起来，Sb加两个，Sc加一个，可以看图示

Tn是分成四分，因为000和111是一样的

Tx/2 Ty/2 Tn=Ts-Tx-Ty/4



![image-20241017171206094](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017171206094.png)

![image-20241017171826563](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241017171826563.png)

### 在第二扇区

000变成010变成110变成111变成010变成11 0变成000

![image-20241018090207639](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241018090207639.png)

计算时长

和扇区一一样，Tn是Ts-Tx-Ty，计算比较值

![image-20241018090824915](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241018090824915.png)

### 在第三扇区

000变成010变成011变成111变成011变成010变成000

![image-20241018091235600](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241018091235600.png)

### 在第四扇区

000变成001变成011变成111变成011变成001变成000

![image-20241018091423897](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241018091423897.png)

### 在第五扇区

000变成001变成101变成111变成101变成001变成000

![image-20241018091519926](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241018091519926.png)

### 在第六扇区

000变成100变成101变成111变成101变成100变成000

![image-20241018091637646](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241018091637646.png)

### 总结扇区计算时间

​	扇区         		1							2 					3						4							5					6

Ta					Ts-Tx-Ty/4		Ts+Tx-Ty/4		Ts+Tx+Ty/4		Ts+Tx+Ty/4	Ts+Tx-Ty/4		Ts-Tx-Ty/4

Tb					Ts+Tx-Ty/4		Ts-Tx-Ty/4		Ts-Tx-Ty/4		Ts+Tx-Ty/4		Ts+Tx+Ty/4	Ts+Tx+Ty/4

Tc					Ts+Tx+Ty/4		Ts+Tx+Ty/4	Ts+Tx-Ty/4		Ts-Tx-Ty/4		Ts-Tx-Ty/4		Ts+Tx-Ty/4