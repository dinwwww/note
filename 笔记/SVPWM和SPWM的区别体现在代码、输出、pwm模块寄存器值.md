# SVPWM和SPWM的区别体现在代码、输出、pwm模块寄存器值



记录2024.12.24 黄革鼎 13168905441





## 生成spwm输出

构建一个虚拟电角度，其中A输出电角度，B输出电角度+120°，C输出电角度+240°，这样构成一个虚拟电角度输出，那么此时测量输出值，就是某两相电压差为相电压，也就是U与V之间电压差。在输出SPWM初始的时候三路输出都是50的占空比，也就是半个周期值，采用中央对齐模式，那么此时三相端电压同，也就是电压差为0相电压为0，此时三相端电压都是电源电压一般。我们控制都是三个端电压电压。





最大调制宽度一半占空比

![image-20241224141829946](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241224141829946.png)

![image-20241224141321860](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241224141321860.png)







此时Uduty就是spwm的输出值，也就是一半占空比+-此时的正负量，因此选取一半中心点，就可以调节+-50的这个值压差，就能算到50-（-50）也就是有100的值利用。虽然调制的值都是0-4000

此时输出UVW也就是0-4000满占空比这样输出，但是选取某一个点的时候，取4000减去对应的低点并不是0也就无法满相电压的利用率。

![image-20241224141535339](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241224141535339.png)

此时加上三次谐波的计算，那么就类似于马鞍波的生成，可以发现在一个周期里面有个三倍周期的谐波产生，幅值1/6

![image-20241224142002183](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241224142002183.png)

这两个波形累加起来，但是调制值要减去50占空比，生成马鞍波svpwm波形，调制宽度不占百分之50，此时被削减了调制宽度，此时电压利用率与spwm一样，但是可以把调制宽度加大，不超出pwm寄存器的值

![image-20241224142221335](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241224142221335.png)

假设此时加大spwm调制宽度，也就是Vm=根号3（*1/2Vm）做出对比可以看出 上下限在4000-0,调制宽度加大，然后三相svpwm取相电压，差值可以取到4000-0，那么电压利用率就会上去

![image-20241224142800748](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241224142800748.png)

![image-20241224142707346](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241224142707346.png)

此时用spwm计算出来的寄存器值 U相-V相，调制最大就到3400左右，如果是svpwm加大调制宽度，就可以达到0-4000，所以用600/400,就可以得出15%的利用率提升，和实际理论公式推导一致。