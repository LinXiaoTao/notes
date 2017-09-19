[参考](http://www.gcssloop.com/customview/Color)

[YUV](https://zh.wikipedia.org/wiki/YUV)

### RGB 模型

在 RGB 模型中，颜色由它们的红色，绿色和蓝色表示。两个 RGB 的相乘，为各个通道的值相差，再对满值(比如十进制的 255 )取余。

计算机显示模式

* 24 比特模式 (RGB888)

  每像素 24 位（比特s per pixel，bpp）编码的RGB值：使用三个 8 位无符号整数（0 到 255）表示红色、绿色和蓝色的强度。这是当前主流的标准表示方法，用于[真彩色](https://zh.wikipedia.org/wiki/%E7%9C%9F%E5%BD%A9%E8%89%B2)和 [JPEG](https://zh.wikipedia.org/wiki/JPEG) 或者 [TIFF](https://zh.wikipedia.org/wiki/TIFF) 等图像文件格式里的通用颜色交换。它可以产生一千六百万种颜色组合，对人类的眼睛来说，其中有许多颜色已经是无法确切的分辨。这种模型也称为 **TrueColor**。


* 16 色

  在这种模式中有 16 种基本颜色


* 16 比特模式

  16 比特模式分配给每种原色各为5比特，其中绿色为 6 比特，因为人眼对绿色分辨的色调更精确。但某些情况下每种原色各占 5 比特，余下的1比特不使用。

* 32 比特模式

  实际就是 24 比特模式，余下的 8 比特不分配到像素中，这种模式是为了提高数据输送的速度（32 比特为一个 [DWORD](https://zh.wikipedia.org/wiki/%E5%AD%97)，DWORD 全称为 Double Word，一般而言一个 Word 为 16 比特或 2 个字节，处理器可直接对其运算而不需额外的转换）。同样在一些特殊情况下，如 [DirectX](https://zh.wikipedia.org/wiki/DirectX)、[OpenGL ](https://zh.wikipedia.org/wiki/OpenGL)等环境，余下的 8 比特用来表示像素的透明度（Alpha）

数值的表示:

| 方式    | RGB表示                   |
| ----- | ----------------------- |
| 浮点    | (1.0,0.0,0.0)           |
| 百分百比  | (100%,0%,0%)            |
| 八位数字  | (255,0,0)或#FF0000(十六进制) |
| 十六位数字 | (65535,0,0)             |



### YUV 模型

"Y" 表示 "明亮度" (Luma)，"U" 和 "V" 则是 "色彩"、"浓度"。

![图像中的 Y，U，和 V](https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Barn-yuv.png/150px-Barn-yuv.png)

为节省带宽起见，大多数的 YUV 格式平均使用的每像素位数都少于 24 位元。

YUV 可以转换为 RGB。



### Android支持的颜色模式

| 颜色模式     | 备注                         |
| -------- | -------------------------- |
| ARGB8888 | 四通道高精度 (32位)，表示每个通道均用八位来表示 |
| ARGB4444 | 四通道低精度 (16位)，表示每个通道均用四位来表示 |
| RGB565   | 屏幕默认模式 (16位)               |
| Alpha8   | 仅有透明通道 (8位)                |

#### 以ARGB8888为例子介绍通道定义：

| 类型       | 解释   | 0(0x00) | 255(0xff) |
| -------- | ---- | ------- | --------- |
| A(Alpha) | 透明度  | 透明      | 不透明       |
| R(Red)   | 红色   | 无色      | 红色        |
| G(Green) | 绿色   | 无色      | 绿色        |
| B(Blue)  | 蓝色   | 无色      | 蓝色        |

**当 RGB 全取最小值 (0或0x000000) 时颜色为黑色，全取最大值 (255或0xffffff) 时颜色为白色**



### Alpha compositing（Alpha 混合）

在计算机图形学中，alpha composition 是将图像与背景组合，以创建部分或完全透明外观的过程。