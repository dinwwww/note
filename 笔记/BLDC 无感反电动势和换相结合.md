# BLDC 无感反电动势和换相结合

首先反电动势这块可以根据图示得出，假设UV导通，那么W就会产生过零点，一种思路就是判断过零信号的组合，通过这个组合来决定下一次换相顺序



![image-20250515182331667](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250515182331667.png)

结合这个图示可以看到，当电角度处于300°-360°的时候，也就是判断到101这个过零信号，那么我就在判断出这个切换状态后，30°电角度，此时大概就是0°电角度，也就是我会导通U+V- 同理，我判断到100的时候，也就说明我电角度刚好30°我会打开U+W-，依次类推循环下去最后构成循环

![image-20250515182447423](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250515182447423.png)

首先连续采样这个比较器状态，每次进入就采样一次

g_motor1_hallless.edge_filter_result =g_motor1_hallless.edge_queue[0]&FILTER_LONG;
	if(g_motor1_hallless.edge_filter_result == FILTER_LONG)
	{
		g_motor1_hallless.queue_filter[0]=1;
	}
	else if (g_motor1_hallless.edge_filter_result == 0x00)
	{
		g_motor1_hallless.queue_filter[0]=0;
	}
	else
	{
		

		g_motor1_hallless.edge_filter_count++;
		return 0;
	}
	
	//V
	g_motor1_hallless.edge_filter_result =g_motor1_hallless.edge_queue[1]&FILTER_LONG;
	if(g_motor1_hallless.edge_filter_result == FILTER_LONG)
	{
		g_motor1_hallless.queue_filter[1]=1;
	}
	else if (g_motor1_hallless.edge_filter_result == 0x00)
	{
		g_motor1_hallless.queue_filter[1]=0;
	}
	else
	{
		g_motor1_hallless.edge_filter_count++;
		return 0;
	}
		
	//W
	g_motor1_hallless.edge_filter_result =g_motor1_hallless.edge_queue[2]&FILTER_LONG;
	if(g_motor1_hallless.edge_filter_result == FILTER_LONG)
	{
		g_motor1_hallless.queue_filter[2]=1;
	}
	else if (g_motor1_hallless.edge_filter_result == 0x00)
	{
		g_motor1_hallless.queue_filter[2]=0;
	}
	else
	{
		g_motor1_hallless.edge_filter_count++;
		return 0;
	}	

其实就是判断此次过零信号

/* 统计过零信号的高电平时间 */
	if(g_motor1_hallless.edge_filter_status == 0)
	{
		if (count_i >= 4)
		{
		
		
		}
		
		g_motor1_hallless.edge_filter_delay=count_j/6;
		g_motor1_hallless.edge_filter_count=0;
		count_j = 0;                        /* 清空时间，以便下一次计算速度 */
	    count_i++;                          /* 捕捉过零点，累计次数 */
	}
	
	if (g_motor1_hallless.edge_filter_status == 1)
	{
	    g_motor1_hallless.edge_filter_count = 0;
	    count_j = 0;
	}
	
	if (g_motor1_hallless.edge_filter_status == 2)  /* 过零信号没变化 */
	{
	    g_motor1_hallless.edge_filter_count++;    /* 不换相时间累计 超时则判定速度为0 */
	
	    if (g_motor1_hallless.edge_filter_count > 15000)
	    {
	        g_motor1_hallless.edge_filter_count = 0;
	        g_motor1_hallless.speed_rpm = 0;  /* 超时换向 判定为停止 速度为0 */
	    }
	}
	
	//检测稳定过0信号
	if(count_i >=4)
	{
		count_i=4;
		edge_flag++;
		if(edge_flag>50)
		{
			edge_flag=50;
			//判断过零信号组合
			g_motor1_hallless.bemf_current_state=(g_motor1_hallless.queue_filter[0])|\
			(g_motor1_hallless.queue_filter[1]<<1)|(g_motor1_hallless.queue_filter[2]<<2);
			//当前过零信号与上一个时刻不同
			if (g_motor1_hallless.bemf_current_state != g_motor1_hallless.bemf_previous_state)
			{
				g_motor1_hallless.edge_filter_total_count++;
				
			}
			if (g_motor1_hallless.edge_filter_total_count >=g_motor1_hallless.edge_filter_delay )
			{
				g_motor1_hallless.edge_filter_total_count=0;
				//再判断过零信号是否改变
				if (g_motor1_hallless.bemf_current_state != g_motor1_hallless.bemf_previous_state)
				{
					if(g_motor1_state.direction == DirectionCCW && g_zero_ctr_status ==3 )
					{
						
						g_motor1_state.position++;
						pfunclist_m1[g_motor1_hallless.bemf_current_state-1]();
					
					}
					else if (g_motor1_state.direction == DirectionCW && g_zero_ctr_status ==3)
					{
						g_motor1_state.position--;
						pfunclist_m2[g_motor1_hallless.bemf_current_state-1]();
					}
					else 
					{
						return 0;
					}


​					

最后起作用是这个

pctr pfunclist_m1[6] =
{
    /* 按过零信号（513264）换向 顺序如下 */
    /* 逆时针换相顺序：(U+V-)->(U+W-)->(V+W-)->(V+U-)->(W+U-)->(W+V-) */
    /* 逆时针对应的过零信号：513264（5(U+V-)1(U+W-)3(V+W-)2(V+U-)6(W+U-)4(W+V-)） */
    &m1_uhwl, &m1_vhul, &m1_vhwl,
    &m1_whvl, &m1_uhvl, &m1_whul
};

/* 顺时针旋转过零信号换向表（逆时针旋转顺序相反）*/
pctr pfunclist_m2[6] =
{   
    /* 546231 */
    /* 按过零信号（513264）换向 顺序如下 */
    /* 5(W+V-)1(U+V-)3(U+W-)2(V+W-)6(V+U-)4(W+U-) */
    /* 顺时针对应的过零信号：546231（5(W+V-)->4(W+U-)->6(V+U-)->2(V+W-)->3(U+W-)->1(U+V-)） */
    &m1_uhvl, &m1_vhwl, &m1_uhwl,
    &m1_whul, &m1_whvl, &m1_vhul       
};

记得要构造一个虚拟函数

void m1_uhvl() { Bldc_SetPhaseState(PHASE_UH_VL, g_motor1_state.pwm_duty); }
void m1_uhwl() { Bldc_SetPhaseState(PHASE_UH_WL, g_motor1_state.pwm_duty); }
void m1_vhwl() { Bldc_SetPhaseState(PHASE_VH_WL, g_motor1_state.pwm_duty); }
void m1_vhul() { Bldc_SetPhaseState(PHASE_VH_UL, g_motor1_state.pwm_duty); }
void m1_whul() { Bldc_SetPhaseState(PHASE_WH_UL, g_motor1_state.pwm_duty); }
void m1_whvl() { Bldc_SetPhaseState(PHASE_WH_VL, g_motor1_state.pwm_duty); }