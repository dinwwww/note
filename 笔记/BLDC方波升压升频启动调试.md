# BLDC方波升压升频启动调试

## 基础配置

同FOC 采用中央对齐模式 PWM2模式 重复计次模式 频率定在10K 且直接使用定时器中断，不用定时器4触发ADC再进入中断，直接在定时器中断内部采集ADC

![image-20250428105551714](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250428105551714.png)

![image-20250428105600508](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250428105600508.png)

写六步换相，每一相的驱动函数，因为六步换相只有两相导通，那么另外一相是不导通，传统方式下桥采用gpio通断策略，但是gpio会导致速率上不去，因此可以采用定时器通道使能与失能方式，这样可以完成两相导通，第三相直接失能 ，但是函数过于冗余，目前还未找到好的解决办法优化代码量，暂时不做考虑

![image-20250428105755159](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250428105755159.png)

![image-20250428105749458](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250428105749458.png)

## 六步换相的启动调制

在启动过程，首先将电机强行拖到，锁定一个状态，例如sept1 ，设定一个时间与占空比大小，这样可以使得电机位移过去，然后进入升压升频策略，在每次更换频率的时候，占空比也要改变，这样最后进入稳定的换向状态



#define START_VOLTAGE_REF 		300 //启动参考电压占空比 最大3200
#define START_FREQ_REF 			300 //启动锁死参考时间 单位100us

#define START_FREQ_DVI 			5 //启动降频率次数 每次衰减20%
#define START_FREQ	 	 		68//平稳后换相频率参考时间 单位100us


static void Bldc_Loop(void)
{
  if(g_motor1_state.run_flag == 1)
  {
    switch (g_zero_ctr_status)
    {
    case 0:
      g_motor1_vvvf_start.voltage_ref=START_VOLTAGE_REF;//启动参考电压占空比
      g_motor1_state.pwm_duty = g_motor1_vvvf_start.voltage_ref;
      Bldc_SetPhaseState(PHASE_UH_VL, g_motor1_state.pwm_duty);
      g_motor1_vvvf_start.freq_t_ref=START_FREQ_REF;
      g_motor1_vvvf_start.delay_temp = 0;  
      g_zero_ctr_status = 1;
      g_motor1_vvvf_start.count=0;
      break;
    

      case 1:
      {
        g_motor1_vvvf_start.delay_temp++;
        if(g_motor1_vvvf_start.delay_temp >=g_motor1_vvvf_start.freq_t_ref)
        {
          g_motor1_vvvf_start.delay_temp=0;
          g_zero_ctr_status=2;
        }
        break;
    
      }
    
      case 2:
      {
    		
    	  g_motor1_vvvf_start.freq_t_ref-=g_motor1_vvvf_start.freq_t_ref/START_FREQ_DVI;//降频启动
          g_motor1_vvvf_start.count++;
    	  if( g_motor1_vvvf_start.count>=10)
    	  {
    		 g_motor1_vvvf_start.count=10;
    	  }
    		BldcChangeVoltage();//升压策略
    	 
        if(g_motor1_vvvf_start.freq_t_ref<=START_FREQ)
        {
          g_motor1_vvvf_start.freq_t_ref=START_FREQ;
          g_zero_ctr_status=1;
        }
        else
        {
          g_zero_ctr_status=1;
        }
    
        g_motor1_vvvf_start.vvvf_count++;
        if(g_motor1_vvvf_start.vvvf_count==6)
        {
          g_motor1_vvvf_start.vvvf_count=0;
        }
    	 BldcSixPhaseSwitching();
    	break;
      }
    
    default:
      break;
    }




  }

}

在这个调制过程中，有几点比较重要，首先固定频率或者占空比，用强拖策略，将电机开环换相跑起来，此时调整某一个，例如就调整占空比大小，看在这个频率下，多少占空比能完美出过零信号，这样就慢慢调到最佳占空比，同样，在这个方法下调频率，最后固定出一组最佳占空比和频率，然后在这个基础上调节开始的时候锁某一项的时间，这样就可以降低启动电流，根据负载的大小可以调节启动的时候时间和电压，降低启动电压，然后进入变频环节，慢慢调试，尝试不同降压策略，怎么样的降压策略可以满足启动出现完整过零信号，确定好降压策略，就可以确定出来变频策略，这个变频策略要灵活，不能太多，否则电机启动的时候会卡克，最后固定出一组参数就可以出现完美信号

![3029320d1a8065576d1a3b48e1f6b71](C:\Users\ding\Documents\WeChat Files\wxid_u4pvjwgc5whb22\FileStorage\Temp\3029320d1a8065576d1a3b48e1f6b71.jpg)





在图中，每个电角度周期都对应六步换相，本次设定六部换相总时间是6.8ms*6≈40ms，利用示波器可以看到，刚好对应每60°一个步，有两个60°完全不开启，因此就有低电平，两个60°完全开启，然后还有两个60°从0v到24v的升压策略，因此刚好对上电角度周期



图示中还有二极管续流现象，会在过零信号检测中过滤掉

下面分析二极管续流现象

当二极管从导通状态切换到截止状态时，会存在一个过渡期，这段时间被称为**反向恢复时间**。在这个期间，二极管内部的载流子（电子和空穴）没有立即消失，而是需要一段时间来恢复。具体过程如下：

- **导通状态下：** 二极管正向导通时，电子从阴极流向阳极，空穴从阳极流向阴极，形成电流。
- **切换到反向：** 当电流开始切断（例如开关切换），二极管依然存在部分载流子（电子和空穴），这些载流子需要一定时间才能消失。
- **反向恢复时间：** 在这个过程中，二极管依然会允许电流反向流动，尽管其电压已变为反向电压。这个现象会持续一定的时间，导致二极管继续导电，即便电源已被切断。

这个反向恢复时间通常是由二极管的物理结构、材料（如硅、砷化镓等）以及外部电路条件决定的。快速恢复二极管（如肖特基二极管）具有较短的恢复时间，而标准二极管（如硅二极管）则恢复时间较长。

大白话来说，二极管截止的时候，通道还存在，因此反向的电压会导通一会，也就解释为什么会有24V信号那么大的压差