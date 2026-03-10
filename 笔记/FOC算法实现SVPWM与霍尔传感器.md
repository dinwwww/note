# FOC算法实现SVPWM与霍尔传感器

## SVPWM配置高级定时器TIM1

13168905441 黄革鼎

在配置的时候，需要使用定时器的中央对齐模式1，并且使用PWM模式2，在这种情况下，CNT的值大于CCR值时候，为有效电平，此时选择有效电平是高电平，那么在重复计次事件为1的情况下，此时在第二个溢出事件后会触发ADC中断，在这个ADC中断的时候，MOS上桥全是关闭状态，采集到电流才是有效值，计算FOC所需三相电流

TIM_TimeBaseStructInit(&TIM_TimeBaseInitStructure);
	/* 时间分频系数 */
    TIM_TimeBaseInitStructure.TIM_ClockDivision=TIM_CKD_DIV1;
    TIM_TimeBaseInitStructure.TIM_CounterMode=TIM_CounterMode_CenterAligned1;
    TIM_TimeBaseInitStructure.TIM_Period=TIMX_SVPWM_PERIOD-1;
    TIM_TimeBaseInitStructure.TIM_Prescaler=0;
    /* 使用重复计次模式 */
    TIM_TimeBaseInitStructure.TIM_RepetitionCounter=1;
    TIM_TimeBaseInit(TIMX_SVPWM_M1, &TIM_TimeBaseInitStructure);
	TIM_ClearFlag(TIMX_SVPWM_M1,TIM_FLAG_Update);
    /* 设置OC模式 */
    TIM_OCStructInit(&TIM_OCInitStructure);
    TIM_OCInitStructure.TIM_OCMode=TIM_OCMode_PWM2;
    /* 互补通道输出使能 */
    TIM_OCInitStructure.TIM_OutputState=TIM_OutputState_Enable;
    TIM_OCInitStructure.TIM_OutputNState=TIM_OutputNState_Enable;
    TIM_OCInitStructure.TIM_OCPolarity= TIM_OCPolarity_High ;
    /* 开启互补功能自动取反PWM模式,猜测已经验证————————————————*/
    TIM_OCInitStructure.TIM_OCNPolarity=TIM_OCNPolarity_High;
    /* 刹车状态电平 */
    TIM_OCInitStructure.TIM_OCIdleState=TIM_OCIdleState_Reset;
    TIM_OCInitStructure.TIM_OCNIdleState=TIM_OCNIdleState_Reset;
    /* CCR比较值默认给1/2 */
    TIM_OCInitStructure.TIM_Pulse=(TIMX_SVPWM_PERIOD>>1);

    /* 配置OC通道参数，使能影子寄存器 */
    TIM_OC1PreloadConfig(TIMX_SVPWM_M1, TIM_OCPreload_Enable);
    TIM_OC1Init(TIMX_SVPWM_M1, &TIM_OCInitStructure);
    
    TIM_OC2PreloadConfig(TIMX_SVPWM_M1, TIM_OCPreload_Enable);
    TIM_OC2Init(TIMX_SVPWM_M1, &TIM_OCInitStructure);
    
    TIM_OC3PreloadConfig(TIMX_SVPWM_M1, TIM_OCPreload_Enable);
    TIM_OC3Init(TIMX_SVPWM_M1, &TIM_OCInitStructure);

并且在给PWM周期的时候，需要在原本的周期上*2，因为使用中央对齐模式，会被分频

在触发ADC中断的时候，选择的不是CCR4的比较事件触发，溢出事件触发，正常是采用CCR4比较事件与

更新事件都可以看外设ADC是否支持这个事件，在选择事件的时候，需要输出这个语句

TIM_SelectOutputTrigger(TIM1, TIM_TRGOSource_Update);

在ADC注入通道选择触发事件需要

	ADC_ExternalTrigInjectedConvConfig(ADC_ADCX,ADC_ExternalTrigInjecConv_T1_TRGO);
	ADC_ExternalTrigInjectedConvEdgeConfig(ADC_ADCX,ADC_ExternalTrigInjecConvEdge_Rising);

## 使用定时器异或模式触发中断，判断霍尔状态TIM5

在定时器特殊模式中，有一个异或模式，刚好可以满足霍尔状态的判断，在每次信号改变的时候进入中断，读取当前三个霍尔IO口的状态就能判断电机电角度在哪个扇区

配置GPIO的时候需要注意采用复用模式，不用上拉与下拉电阻

GPIO_InitTypeDef GPIO_InitStructure;
    /* HAL传感器时钟开启 */
    HAL1_U_GPIO_CLK_ENABLE();
    HAL1_V_GPIO_CLK_ENABLE();
    HAL1_W_GPIO_CLK_ENABLE();
    /* 霍尔传感器引脚，配合TIM5输入捕获异或模式，且输入捕获进入中断 */
    GPIO_StructInit(&GPIO_InitStructure);
    GPIO_InitStructure.GPIO_Pin = HAL1_U_GPIO_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF;
    GPIO_InitStructure.GPIO_OType = GPIO_OType_PP;
    GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_NOPULL;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_100MHz;
    GPIO_Init(HAL1_U_GPIO_PORT,&GPIO_InitStructure);

    GPIO_InitStructure.GPIO_Pin = HAL1_V_GPIO_PIN;
    GPIO_Init(HAL1_V_GPIO_PORT,&GPIO_InitStructure);
    
    GPIO_InitStructure.GPIO_Pin = HAL1_W_GPIO_PIN;
    GPIO_Init(HAL1_W_GPIO_PORT,&GPIO_InitStructure);
    
    GPIO_PinAFConfig(HAL1_U_GPIO_PORT,HAL1_U_GPIO_PINSOURCE,HAL1_U_AF);
    GPIO_PinAFConfig(HAL1_V_GPIO_PORT,HAL1_V_GPIO_PINSOURCE,HAL1_V_AF);
    GPIO_PinAFConfig(HAL1_W_GPIO_PORT,HAL1_W_GPIO_PINSOURCE,HAL1_W_AF);

在配置TIM5的时候，需要注意配置输入捕获通道，选择通道1，滤波根据情况所选，滤波的本质是一段时间内的跳变不计入如上升沿，这个时间和定时器的分配系数与频率有关，触发的话选择双边沿触发，因为上升沿与下降沿都存在。，有个TIM_ICPrescaler是指捕获到的值再分频传入

/* 输入捕获通道配置 */
    TIM_ICStructInit(&TIM_ICInitStructure);
    TIM_ICInitStructure.TIM_Channel=TIM_Channel_1;
    TIM_ICInitStructure.TIM_ICFilter=0xf;
    TIM_ICInitStructure.TIM_ICPolarity=TIM_ICPolarity_BothEdge;
    TIM_ICInitStructure.TIM_ICPrescaler=TIM_ICPSC_DIV1;

TIM_ICSelection_DirectTI这个是之间连接到TI1，因为异或模式本质上是输入捕获，只是把三个通道连接到通道一用通道1去捕获这个边沿，这里面很有讲究



首先要使能这个异或模式

/* 使能霍尔传感器接口，异或映射,TIMx_CH1、CH2 和 CH3 引脚连接到 TI1 输入（异或组合） */
    TIM_SelectHallSensor(TIMX_HAL,ENABLE);

有两种方法实现这个输入的捕获

第一种就是TI1直接连接到TI1FP1输出到IC1，需要使用这句话

/* TI1边沿检测器直接输出TI1FP1*/
    TIM_SelectInputTrigger(TIMX_HAL,TIM_TS_TI1FP1);

那么此时是关闭TIM的从模式，因为没有用到TRC这个模式但是使用这个模式也可以完成，如图所示，在这个模式中滤波之后，启用TRC这个模式

那么通道连接要选择	TIM_ICInitStructure.TIM_ICSelection=TIM_ICSelection_TRC

这样就IC就和TRC关联上，然后经过输入捕获的滤波输出通过TI1F_ED这个滤波输出到这个TRC，TRC再传回来

这就要启用定时器的主从模式    与这两个关键代码有关

TIM_SelectSlaveMode(TIMX_HAL, TIM_SlaveMode_Reset);
//	TIM_SelectMasterSlaveMode(TIMX_HAL,ENABLE);

![image-20241121165722001](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241121165722001.png)

在选择触发中断的时候，可以选择CC1事件，因为没有分频，那么触发一次CC事件就会进入输入捕获的中断 ，此时硬件电路会自动把CNT值更新到CCR1寄存器里面，因为这部分的内部时钟一直在更新如果不采用内部时钟，那么就不会CNT自增，CNT的自增与定时器分频和周期有关

![image-20241121170255049](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241121170255049.png)

因此中断会有溢出中断和CC1中断，在触发CC1中断的时候，也会触发溢出中断，因此当边沿捕获长时间没有进入中断的时候，会一直触发溢出中断，当一定累计值之后就可以检测出堵转

![image-20241121170409256](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241121170409256.png)

if(TIM_GetITStatus(TIMX_HAL,TIM_IT_Update) != RESET)
		{	
			if(TIM_GetITStatus(TIMX_HAL,TIM_IT_CC1) != SET)
		{	
				/* 堵转检测  暂时用10s*/
				TIM_ClearFlag(TIMX_HAL,TIM_IT_Update);
				TIM_ClearFlag(TIMX_HAL,TIM_IT_CC1);
				g_hall_parameter.over_time++;
				if(g_hall_parameter.over_time>=100)
				{
					g_hall_parameter.over_time=0;
					g_hall_parameter.value=bsp_HAL_GetState();
						if(g_hall_parameter.value == 5)
					{
						g_hall_parameter.angle=0.75f;
						

					}
					
					else if(g_hall_parameter.value == 4)
					{
						g_hall_parameter.angle=0.916f;
						
					}
	
					else if(g_hall_parameter.value == 6)
					{
						
						g_hall_parameter.angle=0.083f;
						
					}
					
					else if(g_hall_parameter.value == 2)
					{
						
						g_hall_parameter.angle=0.25f;	
						
					}
					
					else if(g_hall_parameter.value == 3)
					{
						
						g_hall_parameter.angle=0.417f;
					}
					
					else if(g_hall_parameter.value == 1)
					{
						g_hall_parameter.angle=0.583f;
						
					}
					
				}
		}		
		}

正常进入CC1中断

if(TIM_GetITStatus(TIMX_HAL,TIM_IT_CC1) && TIM_GetITStatus(TIMX_HAL,TIM_IT_Update)!= RESET)
	此时计算CNT的值就可以计算出电机在上一次进入CC1中断与这次进入CC1中断的时间，用固定的电角度与固定的时间，那么就可以获得出这段时间的平均速度，用这个平均速度在ADC中断里面累计递增也就是一个积分值，就可以均匀的获得电角度

**首先每次进入中断都会修正这个电角度，这个电角度有说法，目前不确定是否正确**

因为在st的参考代码中，θ角度是q轴与α的一个角度，那么在触发5霍尔值这个边沿的时候，就q轴与α对齐，此时电角度就是0°

但是我们常用的θ是q轴与α的一个角度差，那么在5霍尔值这个边沿的时候，此时d轴少了q轴90°也就是270°

![image-20241121171018639](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241121171018639.png)



![image-20241121171044917](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241121171044917.png)



![image-20241121171124277](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241121171124277.png)

同步电角度测算方法

![image-20241121171201250](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241121171201250.png)



![image-20241121171243837](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241121171243837.png)



**目前的现象就是，采用这个说法，5霍尔值是270电角度，计算出来可以正常旋转，且生成正弦波的电流，SVPWM输出也是马鞍波没问题，但是改成0度电机无法旋转。但是在调1这个PID情况的时候q给定1，反馈回来的值会跳动到负数，目前情况未知**

以下是插值法估计这个电角度，84000000是这个定时器的频率，也就是60度/时间*每个多久进入一次中断，也就是均分这值，然后在中断里面累加，正转的话这个值就增，反转的话这个值就递减取负

g_hall_parameter.angle_add = 0.167f*84000000.0f/10000.0f/(float)g_hall_parameter.temp;

计算角度

g_hall_parameter.angle+=g_hall_parameter.angle_add;

## 锁相环滤波电角度

锁相环大致就是假定一个频率的周期变化的值，然输入与输出作差，再经过PID运算，再经过一个低通滤波，就可以使得这个输出锁住输入的值，使得变化不明显频率相同，但具体代码如何实现还不知道，目前简单锁相环，可以使用

	#define PWM_FRE 					10000
	#define TS 							(1/10000)
	#define OMEGA_FRE 					(100*6.28f)
	#define ANGLE_PLL_KP 				(2*OMEGA_FRE*256*0.8/PWM_FRE)
	#define ANGLE_PLL_KI 				(OMEGA_FRE*OMEGA_FRE*256/PWM_FRE*TS)
	
	v->pll_Err=v->angle-v->pll_angle;
	if(v->pll_Err>=0.5f)
	{
		v->pll_Err-=1.0f;
	
	}
	else if (v->pll_Err<=-0.5f)
	{
		v->pll_Err+=1.0f;
	}
	
	float kp_value = (v->pll_Err *ANGLE_PLL_KP)/256;
	v->pll_ki_sum +=(v->pll_Err *ANGLE_PLL_KI)/256;
	v->pll_angle_speed = kp_value+v->pll_ki_sum;
	
	v->pll_angle+=v->pll_angle_speed;
	if(v->pll_angle> 1.0f)
	{
		v->pll_angle-= 1.0f;
	
	}
	if(v->pll_angle< 0)
	{
		v->pll_angle+=1.0f;
	
	}

# SVPWM

这段计算SVPWM的公式与推导出来不同，不清楚最后为什么加减半个周期，不过这段计算可以省略一段计算步骤，并且抛弃这个反馈回来的v—bus母线电压

	#define  	VOLTAGE_V   		24.0f	//标准工作电压   24V 放大10倍
	#define 	SQRT_OF_3      	 0.57735f
	#define 	SVPWM_KM       	(VOLTAGE_V*SQRT_OF_3)
	#define 	SVPWM_KM_BACKW   0.0721688f   // 倒数  1/(Standard_Work_Voltage_V*Sqrt_OF_3)
	#define     TIMX_SVPWM_PERIOD_HALF             (TIMX_SVPWM_PERIOD/2)


​	
​	/* Ualpha Ubeta 赋值 */
​	v->Ualpha = _Ualpha;
​	v->Ubeta = _Ubeta;
​	
	v->temp1 = v->Ubeta;
	v->temp2 = v->Ubeta*0.5f + v->Ualpha*0.8660254f ;
	v->temp3 = v->temp2-v->temp1;
	
	v->sector=3;
	v->sector=(v->temp2> 0)?( v->sector-1):v->sector;
	v->sector=(v->temp3> 0)?( v->sector-1):v->sector;
	v->sector=(v->temp1< 0)?(7-v->sector) :v->sector;
	
	if(v->sector==1 || v->sector==4)   // 根据矢量扇区计算矢量占空比Tabc
   {
		v->Ta= v->temp2;
		v->Tb= v->temp1-v->temp3;
		v->Tc=-v->temp2;
   }
   else if(v->sector==2 || v->sector==5)
   {
		v->Ta= v->temp3+v->temp2;
		v->Tb= v->temp1;
		v->Tc=-v->temp1;
   }
   else if(v->sector==3 || v->sector==6)
   {
	  v->Ta= v->temp3;
      v->Tb=-v->temp3;
      v->Tc=-(v->temp1+v->temp2);
   }
   else     //异常状态下的判断出的扇区 0---7或者其他就执行0电压矢量
   {
		v->Ta=0;
		v->Tb=0;
		v->Tc=0;
   }

	v->svpwm1=(int16_t)(v->Ta*SVPWM_KM_BACKW*TIMX_SVPWM_PERIOD_HALF)+TIMX_SVPWM_PERIOD_HALF;
	v->svpwm2=(int16_t)(v->Tb*SVPWM_KM_BACKW*TIMX_SVPWM_PERIOD_HALF)+TIMX_SVPWM_PERIOD_HALF;
	v->svpwm3=(int16_t)(v->Tc*SVPWM_KM_BACKW*TIMX_SVPWM_PERIOD_HALF)+TIMX_SVPWM_PERIOD_HALF;

## 调试

万变不离其宗，在FOC这个过程中，采样，换算，电角度，PID，这就是这一套核心，其他的一切都是在这个基础上优化，其中电角度的获取与ADC采样是FOC的根本，用这个重构出一个物理磁链世界，并且DQ轴中，Bin是指本身转子生成的磁场，也就是Bd，然后Bout才是定子产生的，那么如果解耦的话要使得这个面积最大，也就是要使得Q尽量大D等于0，θ就是电角度DQ轴解耦了，力矩就是这个矩形的面积，可以看到如果给d给正值其实也会形成面积，也就能旋转起来

![image-20241121172919051](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241121172919051.png)