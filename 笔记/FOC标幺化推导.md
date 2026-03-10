# 标幺化推导以及代码实现

2025.1.10 黄革鼎 13168905441

## 基准值的选取

所有的基准都是根据电压基准、电流基准、和频率基准来转换得到的

Ubase=Udc/√3 相电压峰值，电压矢量最大值

Ibase=I硬件采集最大电流，控制最大电流

Zbase=Ubase/Ibase

fbase=电机额定频率，电角速度频率基准

wbase=2πfbase

φbase磁链=Ubase/wbase=Ubase/2πfbase

磁链推导过程

磁链的定义

根据法拉第[电磁感应定律](https://baike.baidu.com/item/电磁感应定律/0?fromModule=lemma_inlink)，当磁通随时间变化时，在线圈中将产生[感应电动势](https://baike.baidu.com/item/感应电动势/9499837?fromModule=lemma_inlink)；该[电动势](https://baike.baidu.com/item/电动势/0?fromModule=lemma_inlink)e等于磁链随时间变化率的负值：e=-dΨ/dt式中e与*Ψ*的方向的选取符合右手螺旋关系。可以理解成，当磁链发生变化的时候，通电线圈两端会有感应电势差，那么在理想情况下，磁链的推导可以看成这样，E=-（dφ1/dt+dφ2/dt+dφ3/dt+dφ4/dt+dφ5/dt+），这种是在理想情况下，假设每个磁链都穿透5捆线圈，那么λ=Nφ，因此磁链方程可以推导出E=-dλ/dt

![image-20250110101208400](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110101208400.png)

此时再推导磁链基准值

E=-dλ/dt，根据公式1，可以得到φt=λsin（2πft），对t求导-2πfλ*cos（2πfλ）

当cos值为1时候，φt最大，当φt为最大的时候是看作Vbase，那么Vbase=2πfλ，此时λ看做磁链的基准值，那么λbase=Vbase/2πf

![image-20250110102138268](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110102138268.png)

![image-20250110102159336](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110102159336.png)

电感的基准λ=L*I

Lbase=λbase/Ibase=Ubase/Ibase*2πf=Zbase/wbase

机械角速度基准Ωbase=wbase/Pn

Tbase=Sbase/wbase

![image-20250110104042186](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110104042186.png)

dq轴电压方程标幺化

Ud=Rid+Ld *dId/dt-Welqiq

Uq=Riq+Lq*dIq/dt+We(LdId+φt)

![image-20250110110551735](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110110551735.png)

![image-20250110111541649](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110111541649.png)

![image-20250110110911204](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110110911204.png)

对采样时间进行标幺化

i*dn+1-i*dn，就是把d这个离散化，作差求平均值等号移到右边

![image-20250110111143414](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110111143414.png)

化简

![image-20250110111218464](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110111218464.png)

在把Ts标幺化，Tbase=1/wbase

Ts*就是标幺值，那么此时，Tbase=1/wbase要满足条件

![image-20250110111928029](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110111928029.png)

电流环PI标幺化







![image-20250110112105884](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110112105884.png)

如图，0.72463f是 本系统所用10A/Ubase/√3，也就是统一乘上Ibase/Ubase系数，那么就可以PI参数标幺化，此时如果电流环Ref是0.06那么就可以看成需求输出是0.06*10=0.6A

![image-20250110160447664](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110160447664.png)





锁相环标幺化

![image-20250110112401823](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110112401823.png)

积分的标幺化

![image-20250110112431637](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110112431637.png)

SVPWM的标幺化(猜想)

![image-20250110113338182](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110113338182.png)

在计算SVPWM的时候，幅值都为2/3Udc,但是推导的时候，可以把这个2/3约掉，变成√3/Udc，那么在这个过程中，如果要标幺化，最后就会化简成1，也就是任何系数都不乘以直接算SVPWM公式括号内的



![image-20250110161238189](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110161238189.png)

![image-20250110161347118](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110161347118.png)

在这里，统一带有2/3Udc，但是可以化简成为

![image-20250110161431601](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110161431601.png)

如图√3/Udc，那么带入标幺化系统中，把Ts放到右边，那么就是Tpu=√3/Udc*U1（计算出来的空间合成矢量），那么此时把1/Udc/√3 *  U1=Upu*U1,那么就完成标幺化系统，从另一方面来看，把2/3Udc转换成2/√3

![image-20250110161529381](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250110161529381.png)