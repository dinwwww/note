# 电机模型传递函数推导，电机PID调节电流环理论知识

2024.12.27 黄革鼎 13168905441

![image-20241227141805541](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227141805541.png)

![image-20241227141841905](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227141841905.png)





把稳态部分忽略掉，剩下就是扰动部分的电压方程式







![image-20241227141936943](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227141936943.png)

拉氏变换展开为频域方程 S=d/dt变为时间常数，电阻倒数是增益系数，L/R是可以描述系统的动态响应

![image-20241227142046249](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227142046249.png)

串联一个PI方程

![image-20241227142325029](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227142325029.png)

![image-20241227142258775](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227142258775.png)

得出结果

![image-20241227142503711](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227142503711.png)

最后推导是一个二阶的传递函数，但是为了消除这个零极点影响，要使他降阶



![image-20241227142556417](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227142556417.png)

在这个过程1+S/Ki = (1+Ds)最好凑一个这样形式，这样可以直接消掉分子，那么此时D可以得到

![image-20241227143008777](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227143008777.png)

那么此时Ki可以确定出来是L/R 电路时间常数的倒数，R/L

![image-20241227144336799](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227144336799.png)

T=L/Kp w=Kp/L 那么Kp=Lw=L*2πf因此可以得出

![image-20241227145659643](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227145659643.png)