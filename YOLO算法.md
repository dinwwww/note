# YOLO算法

## yolo1

单阶段，同时输出xywhp，可能速度快，精度低

![image-20260507110434697](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507110434697.png)

### 整体结构

用的AlexNet做基础 和分类网络没有任何区别，只是最后的输出不一样

分辨率越来越下采样，深度越来越大



![image-20260507111226597](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507111226597.png)

### 输出模型解析

全局信息经过卷积池化，输出的信息是7x7x30，这个就等于分成7个块，每个块后面都包含着深度信息以及全局的信息，7x7 映射到原始图片，然后30x1 要解码出来数据

![](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507152416629.png)

![image-20260507155722320](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507155722320.png)

从本质上讲这个框框

因为是根据全局的一个卷积提取，其实每一个框都包含的着全局的信息，那么在这个框里面就会计算置信度，每一个小框的左上角都是相对坐标xy，根据比例缩放，确定这个目标的位置，然后wh也是根据比例进行缩放，全部进行归一化，因为每个框都包含的全局信息，所以在每次学习的时候，会更新这个xywh，满足这个框的范围 cp

![image-20260507160943863](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507160943863.png)

### 置信度

IOU因为前期有标注，那么最后的置信度，就是根据iou比，就是量化的指标，交集和并集的公式比

![image-20260507161407072](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507161407072.png)



这个就是，在预测的时候，这个IOU是模型认为的交并比，如果是我预测的物体，那么就有一个交并比，其实就和xywh一样，需要学习更新，在前面学习的时候慢慢更新出来的一个值，就是模型认为的框出来的置信程度的比值

![image-20260507161554684](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507161554684.png)

在最后，每个小块都有预测值，最后晒出来最好的，并不是拼在一起

![image-20260507162118323](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507162118323.png)

![image-20260507162201076](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507162201076.png)

### 后处理

就是处理模型输出的所有值，每个框的类别的概率，是否有物体的含义，然后认为的交并比，三者乘起来就是对应类别的物体置信度

![image-20260507162520370](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507162520370.png)

如果有多个框

其实就是找到最大的那个框

![image-20260507162642533](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507162642533.png)

### yolo损失函数

如何标注的呢

假设标注成功 xywh c p也有了



### ![image-20260507163604064](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507163604064.png)

坐标损失，先确定有没有置信比 ，然后再计算差值，为什么这个w要开根号，因为y=x这个函数，如果不开根号，大物体的iou可能差值大，但是在数值上反应出来很小，所以开根后，可以看出小物体的检测差值，比大物体差值更大，朝着大框预测值更好的方向去

![image-20260507164103584](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507164103584.png)

![image-20260507164541395](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507164541395.png)

置信度 其实就是实际的iou与预测出来iou更新预测的iou

![image-20260507164650636](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507164650636.png)

### yolov2

![image-20260507181309059](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507181309059.png)

### BN层的添加

![image-20260507181527178](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507181527178.png)

全卷积和anchor框的添加

输出值使得xy不落在格子边缘，

增加独立的p

![image-20260507182659462](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507182659462.png)

![image-20260507182513204](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507182513204.png)

添加一个anchor框

相当于给他一个预测方向刚好对应上输出的9个框，多了框，那就可以预测多目标，并不是只有一个框

![image-20260507183216663](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507183216663.png)

如何获取预先的anchor框

![image-20260507184100449](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507184100449.png)

暂定不学了

### yolov11

CBS模块

引入不同的激活函数

![image-20260507190722417](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507190722417.png)

![image-20260507190756939](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507190756939.png)

相比于relu函数，他会缓解梯度消失的问题

![image-20260507190922243](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507190922243.png)

![image-20260507191024635](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507191024635.png)

### Bottleneck

就是为了降低计算量使得参数量减少

![image-20260507200638377](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507200638377.png)

![image-20260507200031141](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507200031141.png)

![image-20260507200125310](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260507200125310.png)

### SPPF模块

先是由SPP模块改进，通道数可能不一样，但是尺寸和原输入是一样的

先卷积再池化，其实是一样的，但是速度更快

![image-20260508132518029](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260508132518029.png)

![image-20260508132928611](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260508132928611.png)

### C3K2是由C3继承

C3就是由这个ResUnit—F ResUnit—T 这个两个模块构成，主要区分有没有残差连接

![image-20260508133347934](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260508133347934.png)

本质上C3K这个模块就是把ResUnit换成Bottleneck

![image-20260508133715143](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260508133715143.png)

### 注意力机制模块

![image-20260508134054612](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260508134054612.png)

输入分别有qkv，q是和自己的相乘，然后分别给别的输入做交叉信息，k也是自己的提前，但是获取他人的信息q，然后qk两个矩阵相乘，最后再和自己的v矩阵乘，然后所有的这个输出融合起来，做成b，这样每个b包含了所有的输入信息，但是只有原始的输入带有自己的v信息，通过反向传播会更新qkv的w权重，这样就能比对出最佳的

![image-20260508134849565](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260508134849565.png)

![image-20260508135046776](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260508135046776.png)

![image-20260508140258399](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260508140258399.png)

### C2PSA模块

![image-20260508155350239](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260508155350239.png)