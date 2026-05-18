# 传统图像识别车牌python实现 

## 1、图像预处理

预处理就是要把原输入的图片，先做处理，去掉不方便辨别的，找到要识别的主题

### 1、图像灰度化

减少计算量、降低特征维度、简化后续边缘检测 / 分割 / 特征提取

![image-20260427114024241](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260427114024241.png)

在代码体现就是这句话，先转换成灰度图

```
gray = cv2.cvtColor(original_img, cv2.COLOR_BGR2GRAY)
```

2、灰度直方图均衡化

![image-20260427114733068](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260427114733068.png)

### 3.灰度拉伸

就是把原有的灰度图，重新再0-255分配

### 4.图像平滑滤波

这里用的双边滤波

blurred = cv2.bilateralFilter(gray, 11, 17, 17)

![image-20260427114821147](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260427114821147.png)

### 5.图像二值化

**单通道灰度图**变成**只有纯黑 (0) 和纯白 (255) 两种颜色**的图像

```
edges = cv2.Canny(blurred, 30, 200)
```

在代码上体现 30 -200

## 2、车牌定位

### 膨胀

代码上看，把图片转成HSV，在HSV上面做掩码，再在掩码下作膨胀作用

color_mask = get_dilated_color_mask(original_img)

```
valid_edges = cv2.bitwise_and(edges, edges, mask=color_mask)
```

本意也是滤波，用核把白色区域放大，变膨胀

![image-20260427115346882](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260427115346882.png)

### 腐蚀

和膨胀相反

![image-20260427115531846](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260427115531846.png)

![image-20260427115614882](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260427115614882.png)

### 开运算，闭运算



![image-20260427120338871](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260427120338871.png)

在代码上

**第一行**：定义一个**横向更长的 9×5 矩形核**，专门用来连接横向断裂的边缘

**第二行**：执行**闭运算（先膨后腐）**，修复边缘的断裂和缺口，让边缘形成完整闭环

```
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (9, 5))
closed = cv2.morphologyEx(valid_edges, cv2.MORPH_CLOSE, kernel)
```

### 边缘检测

就是计算特征点，几个比较符合的特征点，连起来





![image-20260427120740415](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260427120740415.png)

```
计算最最大面积前十五的轮廓
contours, _ = cv2.findContours(closed.copy(), cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
contours = sorted(contours, key=cv2.contourArea, reverse=True)[:15]
```



## 字符切割

![image-20260427120938514](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20260427120938514.png)

### 车牌倾斜校准

目的是找到倾斜角度θ，然后做倾斜矩阵，反射回去，让车牌变正常

### 去除车牌边框

可以找到父层级，然后提取子层级

### 垂直投影分割

找到字符间隔之间的点，然后计算投影，如果纯黑，发现没有投影，那就是字符中间，直接分割出来



### 归一化

让字体均匀



先去最符合的边缘出来，

做简单的宽高比计算，提出车来车牌的边缘，然后计算校正的矩形，找到矩形的点位，然后从原图中间切割出来

```
rect = cv2.minAreaRect(c)
    box = cv2.boxPoint(rects)
    box = np.int64(box)

    (x_c, y_c), (w, h), angle = rect
    if w == 0 or h == 0: continue

    if w < h: w, h = h, w
    aspect_ratio = w / float(h)

    if 1.8 <= aspect_ratio <= 5.5:
        x, y, bw, bh = cv2.boundingRect(box)
        x, y = max(0, x), max(0, y)

        pad_x, pad_y = int(bw * 0.05), int(bh * 0.05)
        if y + bh - pad_y <= y + pad_y or x + bw - pad_x <= x + pad_x:
            continue

        crop_image = original_img[y + pad_y: y + bh - pad_y, x + pad_x: x + bw - pad_x]
        plate_color = reg_area_color(crop_image)

        if plate_color in ["blue", "green", "yellow"]:
            return copy_img, box, plate_color

return copy_img, None, None
```

## 字符识别

本文用的OCR处理，在裁出车牌区域后，再灰度值处理，送进OCR处理，然后去识别

用自己训练的网络识别不准确，不知道为什么，我自己训练的是LetNet网络，准确率还挺高，但是在这个切割出来的图片就准确率不高，可能训练集和实际的切割出来的图片不一样，还需要调整切割出来的图片，训练出来的模型是可以用的，关键应该是切割出来的字符，要怎么切割丢给模型，因为OCR这个框架已经帮你处理了这部分

​																																				by ding 2026.4.27