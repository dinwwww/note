# ST7735 显示芯片移植记录与BUG

 BLOCK DIAGRAM 框图

使用SPI 主从模式，分别两种，一种是控制指令cmd，另外一种就是写数据寄存器，写控制指令使用正常的收发模式，因为写数据的时候，用到大量数据，所以可以加上DMA，利用DMA的话，就能一次进行大量数据传输，把CPU空下来可以处理别的事情

![image-20250603163507418](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250603163507418.png)

*(uint8_t *)&SPI1->DR = data[i];

在单独写数据的时候，这个是关键，因为我需要每次在SPI的dr寄存器赋值，正常来说是SPI1->DR = data[i]; 但是我每次收发数据只写一位，因此

spi1->DR其实本质上就是一个指针，那么这个指针是(*(SPI_TypeDef *)0x40013000).DR 取的SPI1的基础地址，偏移到DR寄存器这个位置，每次指针自增的步长就是16个字节，因此我赋值的话，加上强转可以让他只取首8字节，且解引用等于把dr寄存器的前八位直接赋值操作



通过DMA形式发数据，那就需要建一个缓存数组，在这个缓存数组里面，要在DMA发送完成中断函数那，写一个标志位判断，否则会造成片选信号提前拉起，因为DMA数据搬运完成，不代表SPI数据发送完成

![image-20250604092051185](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604092051185.png)

定义函数指针

![image-20250604093128183](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604093128183.png)

完成函数指针定义

![image-20250604093147050](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604093147050.png)

通过外部函数的形式注册

![image-20250604093207369](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604093207369.png)

在中断内部调用

![image-20250604093248718](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604093248718.png)

在设定窗口大小的时候，涉及一个大小端对齐问题，需要先发高0位，再发第八位

![image-20250604104319353](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604104319353.png)

![image-20250604105044677](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604105044677.png)

![image-20250604105316389](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604105316389.png)

![image-20250604105343413](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604105343413.png)

因此要手动交换顺序

![image-20250604105417094](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604105417094.png)

指针相减

![image-20250604142503250](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604142503250.png)

![image-20250604142440205](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604142440205.png)

常用的节省空间操作

![image-20250604142749475](C:\Users\ding\AppData\Roaming\Typora\typora-user-images\image-20250604142749475.png)
