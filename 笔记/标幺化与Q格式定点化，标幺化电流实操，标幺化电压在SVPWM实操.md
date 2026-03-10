# 标幺化与Q格式定点化，标幺化电流实操，标幺化电压在SVPWM实操

2024.12.27

在代码中要实现电流环的标幺化首先要计算出来每个ADC计数值对应的电流大小，然后假定一个基准电流，例如本块板子基准电流设定10A，然后再去除这个系数10，得到就是标幺化的结果，要验证是否正确，首先用电流钳把标幺化的值设定到0.5左右，示波器看电流是否5A，能否对应上，如果对应上，那么标幺化是对的。

![image-20241227090429467](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227090429467.png)

在电压中也需要标幺化，我们的母线电压是Udc，这个是线电压，不是相电压，因此需要除以根号3，不过在计算SVPWM的时候已经把这个标幺化结束了，具体看图

![image-20241227091108619](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227091108619.png)

![image-20241227091134478](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20241227091134478.png)