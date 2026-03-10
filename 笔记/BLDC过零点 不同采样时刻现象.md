# BLDC过零点 不同采样时刻现象

前文出现过零采样点得到数据跳变严重，不是因为频率和占空比的原因，因为前文PWM使用PWM2 中央对齐模式，假设U相导通，那我W相互补导通那个时间，可是在互补导通的时候直接用满占空比减去这个时间，也就会造成这个三极管开关现象严重，无论什么情况下，采样点我们都需要避开三极管开闭吸合状态。

首先解释一个现象，那就是三极管续流现象，为什么有时候有，有时候没有，是因为占空比给的太小的话，反电动势无法抬升这个反向电压叠加，因为三极管有个反向导通电压。

那么经过试验验证，也就是把PWM产生方式改成PWM1 向上计数，且每次采样点我都要在上端导通的时间内，或者远离这个上端导通的时间点，也就是不能在这个开关状态附近，在这个点附近，也就是能完成过零采样点，每次进入中断，我都能读取到这个寄存器的状态。

因此改PWM1 向上计数，用通道4触发中断，CCR比较值要根据合适的情况选择

经试验，这样输出值都很完美

 TIM_TimeBaseInit(TIM1, &timInitStruct);

    /* Channel 1, 2,3 and 4 Configuration in PWM mode */
    timOCInitStruct.TIM_OCMode = TIM_OCMode_PWM1;
    timOCInitStruct.TIM_OutputState = TIM_OutputState_Enable;
    timOCInitStruct.TIM_OutputNState = TIM_OutputNState_Enable;
    timOCInitStruct.TIM_Pulse = TIMER1_ARR_HALF_VALUE;
    timOCInitStruct.TIM_OCIdleState = TIM_OCIdleState_Reset;
    timOCInitStruct.TIM_OCNIdleState = TIM_OCNIdleState_Reset;
    
    TIM_OC1Init(TIM1, &timOCInitStruct);
    TIM_OC1PolarityConfig(TIM1, TIM_OCNPolarity_High);
    TIM_OC1NPolarityConfig(TIM1, TIM_OCNPolarity_High);
    
    timOCInitStruct.TIM_Pulse = TIMER1_ARR_HALF_VALUE;
    TIM_OC2Init(TIM1, &timOCInitStruct);
    TIM_OC2PolarityConfig(TIM1, TIM_OCNPolarity_High);
    TIM_OC2NPolarityConfig(TIM1, TIM_OCNPolarity_High);
    
    timOCInitStruct.TIM_Pulse = TIMER1_ARR_HALF_VALUE;
    TIM_OC3Init(TIM1, &timOCInitStruct);
    TIM_OC3PolarityConfig(TIM1, TIM_OCNPolarity_High);
    TIM_OC3NPolarityConfig(TIM1, TIM_OCNPolarity_High);