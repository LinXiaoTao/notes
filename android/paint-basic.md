[参考](http://hencoder.com/ui-1-2/)



## 颜色

`Canvas` 绘制的内容，有三层对颜色的处理：

![color](https://ws3.sinaimg.cn/large/52eb2279ly1fig6dcywn2j20j909yabu.jpg)



### Shader

除了直接设置颜色，`Paint` 还可以使用 Shader（着色器）。

它和直接设置颜色的区别是，着色器设置的是一个颜色方案，或者是一套着色规则。当设置了 `Shader` 之后，`Paint` 在绘制图形和文字时就不使用 `setColor/ARGB()` 设置的颜色了，而是使用 `Shader` 的方案中的颜色。

* LinearGradient 线性渐变着色器

* RadialGradient  辐射渐变着色器

* SweepGradient 扫描渐变着色器

* BitmapShader Bitmap 着色器

* ComposeShader 混合着色器

  > 在硬件加速下是不支持两个相同类型的 `Shader` 



### ColorFilter

为绘制设置颜色过滤。颜色过滤的意思，就是为绘制的内容设置一个统一的过滤策略，然后 `Canvas.drawXXX()` 方法会对每个像素都进行过滤后再绘制出来

* LightingColorFilter 模拟简单的光照效果
* PorterDuffColorFilter 使用一个指定的颜色和一种指定的 `PorterDuff.Mode` 来与绘制对象进行合成
* ColorMatrixColorFilter 使用一个 `ColorMatrix` 来对颜色进行处理



### Xfermode

`Xfermode` 指的是你要绘制的内容和 `Canvas` 的目标位置的内容应该怎样结合计算出最终的颜色。但通俗地说，其实就是要你以 **绘制的内容作为源图像**，以 **View 中已有的内容** 作为目标图像，选取一个 `PorterDuff.Mode` 作为绘制内容的颜色处理方案。

```java
Xfermode xfermode = new PorterDuffXfermode(PorterDuff.Mode.DST_IN);
//离屏缓冲
int saved = canvas.saveLayer(null, null, Canvas.ALL_SAVE_FLAG);
canvas.drawBitmap(rectBitmap, 0, 0, paint); // 画方  
paint.setXfermode(xfermode); // 设置 Xfermode  
canvas.drawBitmap(circleBitmap, 0, 0, paint); // 画圆  
paint.setXfermode(null); // 用完及时清除 Xfermode
canvas.restoreToCount(saved);
```

