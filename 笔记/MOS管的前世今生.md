# MOS管的前世今生

先是晶体管→二极管→三极管→MOS管

## 二极管

在硅钢片上掺杂不同的元素，形成PN半导体，结合处PN结有单向导电性

正向导电

![image-20250416093635045](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416093635045.png)

反向导电

![image-20250416093744750](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416093744750.png)

## 三极管

用三个半导体，组成小电流控制大电流P向N2流向小电流，那么N1 N2可以导通大电流

![image-20250416093901396](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416093901396.png)

如果此时P关断，那么N1 N2无法导通

![image-20250416094031412](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416094031412.png)

由于通过大电流有限，大电流必须通过两层PN结，导致损耗大

![image-20250416094202373](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416094202373.png)

而且三极管是电流控制器件

![image-20250416094314139](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416094314139.png)

修改成新结构

施加电压，电流从N到P流，那么就会增大PN结，就会把这里截断，显示不导通的状态

![image-20250416105000920](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416105000920.png)

负电压断开

![image-20250416105417261](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416105417261.png)

因此这个结构是电压控制期间，在PN结处没有电流通过，称为结型场效应管JFET

D为漏极 S是源极 G是栅极

![image-20250416105609016](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416105609016.png)

![image-20250416105647872](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416105647872.png)

箭头朝内为N型

![image-20250416105757928](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416105757928.png)

箭头朝外是P型

![image-20250416105814648](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416105814648.png)

此类设计又有缺点，夹紧与放开，很难做到完全没有电流通过，因此更改成电容方式

![image-20250416110235697](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416110235697.png)

在P型半导体中，有大量空穴和少量自由电子，施加正向电压的话，自由电子被吸引，施加越大，吸引的电子越多

![image-20250416110443157](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416110443157.png)

一般这样BS连接在一起，一起引出

![image-20250416110557072](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416110557072.png)

这种利用绝缘层二氧化硅，称为增强型MOSFET场效应管MOS管

![image-20250416110945227](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416110945227.png)

![image-20250416110956882](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416110956882.png)

耗尽型mos管

![image-20250416111115890](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416111115890.png)

![image-20250416111130021](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416111130021.png)

![image-20250416111209727](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416111209727.png)



因为N型，是外部施加正电压，这样才导通

![image-20250416111221812](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416111221812.png)



P型是外部施加负电压，这样才导通

![image-20250416111326059](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416111326059.png)

因为有寄生二极管的原因

![image-20250416111529631](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416111529631.png)

![image-20250416111559574](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250416111559574.png)