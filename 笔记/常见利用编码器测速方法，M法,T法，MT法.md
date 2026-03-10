# 常见利用编码器测速方法，M法,T法，M/T法

## M法 频率测量法

在一定时间内，计算编码器脉冲数，计算速度值。

单圈编码器总数为C，在T0时间内，统计到脉冲数量为M0，那么转速n的公式是

![image-20240909094214025](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240909094214025.png)

在高速时，M0增加的很大，可以使计算速率更准确

## 基波频率

在**同步电机**中，电机的转子与旋转的磁场同步转动。这意味着转子的转速与定子产生的旋转磁场的频率是直接相关的。

电机定子绕组中通入的三相交流电会产生一个**旋转磁场**，该旋转磁场的转速称为**同步转速**。

同步转速**与定子绕组的供电频率和电机的**极对数密切相关。

对于一个旋转磁场：

- 如果有**1对极**，旋转磁场在一个周期内只完成一圈旋转。
- 如果有**2对极**，旋转磁场在一个周期内完成两次极性翻转，相当于半圈旋转，因此需要两倍的频率来完成一圈旋转。

因此，最终的**基波频率**与电机同步转速的关系公式为：

![image-20240909133843058](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240909133843058.png)

60固定常数是指

它来自于将**频率的单位（Hz，秒）\**转换为\**转速的单位（RPM，分钟）**，因为每分钟有60秒，而每转相当于两个极的翻转。

因此确定转速需要确定基波

​	

## 利用基波频率与M法测转速

首先基波频率以1000rpm为基准，计算出电角度频率

#define BaseFreq 				(1000L*50/60)

再使用频率的倒数，可以计算出1000rpm为基准的，电角度频率的时间的倒数可以计算出旋转一圈电角度的时间，然后电角度增量0-1的范围内（无符号），乘以这个时间常量，可以计算出这次电角度增量所用时间。

再通过M法计算转速，电角度增量算的时间/时间间隔40us 那么可以算出以1000为基准0-1范围内的转速，单位rpm再用求出来结果乘以1000就可以获得实际转速。

本质上M法测量，就是在一定时间间隔内，编码器增量/时间间隔，也就是角度增量/时间间隔，将他与基波频率联系起来，在高频触发中断范围内，计算电角度增量，以基准频率去求转速比例，可以更适应低速区间（自己总结）。

//时间常数 频率倒数为25Khz 		40us   0.00004s
#define T 						(0.001f/25)				
//分频系数
#define SpeedLoopPrescaler							2（有标志位，两次计算一次电角度增量）
#define SpeedCalPrescaler								2



比例系数，其中K1才有效计算转速以1000rpm为基准，K2K3为低通滤波，截止频率取20

其中K1可以看做，Tbase（1000转的电角度基准时间单位s）/采样间隔时间0.00004（单位s）与电角度增量（0-1范围）乘出来后没有单位了，只是一个比例值

MotorSpeed.K1=(1.0f / (BaseFreq * T * SpeedLoopPrescaler ));
MotorSpeed.K2 =(1.0f/(1.0f + T *SpeedLoopPrescaler * 2 * PAI*20));
MotorSpeed.K3 = (1.0f) - MotorSpeed.K2;

## 代码

void DRV_MotorSpeed(DRV_MotorSpeedTypeDef*v, float ElectricalSpeedDegree)
{
	v->SpeedCalCnt ++;	
	if (v->SpeedCalCnt >=2)
	{
		v->SpeedCalCnt=0;
	

		v->ElecTheta = ElectricalSpeedDegree;
		//计算电角度差值
		v->DeltaTheta = (v->ElecTheta) -(v->OldElecTheta) ;
		//用于下一时刻计算差值
		v->OldElecTheta = v->ElecTheta;
		//判断差值是否超过一圈,于电角度计算同理
		if (v->DeltaTheta > (0.5f))
		{
			v->DeltaTheta = v->DeltaTheta - (1.0f);
		}
		else if (v->DeltaTheta < (-0.5f))
		{
			v->DeltaTheta = v->DeltaTheta + (1.0f);
		}
		//滤波
		v->DeltaTheta = (v->DeltaTheta)*(0.8f) + (v->OldDeltaTheta)*(0.2f);
		v->OldDeltaTheta = v->DeltaTheta;
		//计算转速M法 间隔时间内电机转过的角度
		//当前电角度旋转倒数差值 * 基波时间(1000rpm为基础) / T 可以获得转速
		v->Speed = (v->K1 * v->DeltaTheta);
		//低通滤波截止频率为20
		v->Speed = (v->OldSpeed * v->K2)+(v->Speed * v->K3);
		v->OldSpeed = v->Speed;
		
		//加权滤波
		v->Temp[0] = (v->Speed + v->Temp[1]*3.0f)*0.25f;
		v->Temp[1] = v->Temp[0];
		//实际转速*1000 是以1000为基波频率
		v->Temp[2] = v->Temp[0] * 1000.0f;
		
	}

## 补充，关于电角度速度的思考

![image-20240911094926754](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240911094926754.png)

有关于电角度速度的计算，可以理解成有关于电角度增量关于时间t的线性系统，再对此t取微分，就可以获得电角度速度。但是在实际的系统中，采样是离散采样，不能直接取微分，因为无法取到时间函数t。因此利用线性差值方法求电角度速度。

具体解释如图

![image-20240911095148332](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20240911095148332.png)