[PorterDuff.Mode](https://developer.android.com/reference/android/graphics/PorterDuff.Mode.html)



ProterDuff.Mode 一共有 17 个，可以分为两类：**Alpha 合成（Alpha Compositiing）**和 **混合（Blending）**。


$$
\alpha_{out} 表示结果 alpha 值
$$

$$
C_{out} 表示结果 color 值
$$

源图像和目标图像

![源图像](https://developer.android.com/reference/android/images/graphics/composite_SRC.png)

![目标图像](https://developer.android.com/reference/android/images/graphics/composite_DST.png)



### Alpha Compositing

Alpha 合成，其实就是 「PorterDuff」 这个词所指代的算法。 「PorterDuff」 并不是一个具有实际意义的词组，而是两个人的名字（准确讲是姓）。这两个人当年共同发表了一篇论文，描述了 12 种将两个图像共同绘制的操作（即算法）。而这篇论文所论述的操作，都是关于 **Alpha 通道（也就是我们通俗理解的「透明度」）**的计算的，后来人们就把这类计算称为 **Alpha 合成** ( Alpha Compositing ) 。

* SRC

  源像素替换目标像素

  ![SRC](https://developer.android.com/reference/android/images/graphics/composite_SRC.png)
  $$
  \alpha_{out} = \alpha_{src}
  $$

  ​
  $$
  C_{out} = C_{src}
  $$

* SRC_ATOP

  丢弃不包括目标像素的源像素。在目标像素上绘制剩余的源像素

  ![SRC_ATOP](https://developer.android.com/reference/android/images/graphics/composite_SRC_ATOP.png)
  $$
  \alpha_{out} = \alpha_{dst}
  $$
  $$
  C_{out} = \alpha_{dst} * C_{src} + (1 - \alpha_{src}) * C_{dst}
  $$

* SRC_IN

  保持覆盖目标像素的源像素，丢弃剩余的源像素和目标像素

  ![SRC_IN](https://developer.android.com/reference/android/images/graphics/composite_SRC_IN.png)
  $$
  \alpha_{out} = \alpha_{src} + \alpha_{dst}
  $$

  $$
  C_{out} = C_{src} * a_{dst}
  $$

* SRC_OUT

  保持没有覆盖目标像素的源像素。丢弃覆盖目标像素的源像素。丢弃所有目标像素。

  ![SRC_OUT](https://developer.android.com/reference/android/images/graphics/composite_SRC_OUT.png)

$$
\alpha_{out} = (1 - \alpha_{dst}) * \alpha_{src}
$$

$$
C_{out} = (1 - \alpha_{dst}) * C_{src}
$$

* SRC_OVER

  源像素绘制在目标像素上

  ![SRC_OVER](https://developer.android.com/reference/android/images/graphics/composite_SRC_OVER.png)

$$
\alpha_{out} = \alpha_{src} + (1 + \alpha_{src}) * \alpha_{dst}
$$

$$
C_{out} = C_{src} + (1 - \alpha_{src}) * C_{dst}
$$

* XOR

  丢弃覆盖目标像素的源像素和被覆盖的目标像素。绘制剩余的源像素和目标像素。

  ![XOR](https://developer.android.com/reference/android/images/graphics/composite_XOR.png)

$$
\alpha_{out} = (1 - \alpha_{dst}) * \alpha_{src} + (1 - \alpha_{src}) * \alpha_{dst}
$$

$$
C_{out} = (1 - \alpha_{dst}) * C_{src} + (1 - \alpha_{src}) * C_{dst}
$$

* DST

  丢弃源像素，保留完整的目标像素

  ![DST](https://developer.android.com/reference/android/images/graphics/composite_DST.png)
  $$
  \alpha_{out} = \alpha_{dst}
  $$

  $$
  C_{out} = C_{dst}
  $$

* DST_ATOP

  丢弃没有覆盖源像素的目标像素。在源像素上绘制剩余的目标像素

  ![DST_ATOP](https://developer.android.com/reference/android/images/graphics/composite_DST_ATOP.png)

$$
\alpha_{out} = \alpha_{src}
$$

$$
C_{out} = \alpha_{src} + C_{dst} + (1 + \alpha_{dst}) * C_{src}
$$

* DST_IN

  保留覆盖源像素的目标像素，丢弃剩余的源像素和目标像素

  ![DST_IN](https://developer.android.com/reference/android/images/graphics/composite_DST_IN.png)

$$
\alpha_{out} = \alpha_{src} + \alpha_{dst}
$$

$$
C_{out} = C_{dst} * \alpha_{src}
$$

* DST_OUT

  保留没有覆盖源像素的目标像素。丢弃覆盖源像素的目标像素。丢弃所有的源像素。

  ![DST_OUT](https://developer.android.com/reference/android/images/graphics/composite_DST_OUT.png)

$$
\alpha_{out} = (1 - \alpha_{src}) * \alpha_{dst}
$$

$$
C_{out} = (1 - \alpha_{src}) * C_{dst}
$$

* DST_OVER

  源像素被绘制在目标像素之后

  ![DST_OVER](https://developer.android.com/reference/android/images/graphics/composite_DST_OVER.png)

$$
\alpha_{out} = \alpha_{dst} + (1 - \alpha_{dst}) * \alpha_{src}
$$

$$
C_{out} = C_{dst} + (1 - \alpha_{dst}) * C_{src}
$$

* CLEAR

  源像素覆盖的目标像素背清除为 0

  ![CLEAR](https://developer.android.com/reference/android/images/graphics/composite_CLEAR.png)
  $$
  \alpha_{out} = 0
  $$

  $$
  C_{out} = 0
  $$









### Blending modes

* DARKEN

  保留源像素和目标像素的最小值

  ![DARKEN](https://developer.android.com/reference/android/images/graphics/composite_DARKEN.png)
  $$
  \alpha_{out} = \alpha_{src} + \alpha_{dst} - \alpha_{src} * \alpha_{dst}
  $$

  $$
  C_{out} = (1 - \alpha_{dst}) * C_{src} + (1 - \alpha_{src}) * C_{dst} + min(C_{src}, C_{dst})
  $$

* LIGHTEN

  保留源像素和目标像素的最大值

  ![LIGHTEN](https://developer.android.com/reference/android/images/graphics/composite_LIGHTEN.png)
  $$
  \alpha_{out} = \alpha_{src} + \alpha_{dst} - \alpha_{src} * \alpha_{dst}
  $$

  $$
  C_{out} = (1 - \alpha_{dst}) * C_{src} + (1 - \alpha_{src}) * C_{dst} + max(C_{src}, C_{dst})
  $$

* MULTIPLY

  源像素和目标像素相乘

  ![MULTIPLY](https://developer.android.com/reference/android/images/graphics/composite_MULTIPLY.png)
  $$
  \alpha_{out} = \alpha_{src} * \alpha_{dst}
  $$

  $$
  C_{out} = C_{src} * C_{dst}
  $$

* SCREEN

  源像素和目标像素相加，然后减去源像素和目标像素相乘。

  ![SCREEN](https://developer.android.com/reference/android/images/graphics/composite_SCREEN.png)
  $$
  \alpha_{out} = \alpha_{src} + \alpha_{dst} - \alpha_{src} * \alpha_{dst}
  $$

  $$
  C_{out} = C_{src} + C_{dst} - C_{src} * C_{dst}
  $$

* OVERLAY

  根据目标像素，对源像素和目标像素进行加倍的 MULTIPLY 或者 SCREEN。

  ![OVERLAY](https://developer.android.com/reference/android/images/graphics/composite_OVERLAY.png)
  $$
  \alpha_{out} = \alpha_{src} + \alpha_{dst} - \alpha_{src} * \alpha_{dst}
  $$

  $$
  \begin{equation}
   C_{out} = \begin{cases} 2 * C_{src} * C_{dst} & 2 * C_{dst} \lt \alpha_{dst} \\
   \alpha_{src} * \alpha_{dst} - 2 (\alpha_{dst} - C_{src}) (\alpha_{src} - C_{dst}) & otherwise \end{cases}
   \end{equation}
  $$











### Android 应用

* [ComposeShader](https://developer.android.com/reference/android/graphics/ComposeShader.html)

  ``` jade
  ComposeShader (Shader shaderA, Shader shaderB, PorterDuff.Mode mode)
  ```

* [PorterDuffColorFilter](https://developer.android.com/reference/android/graphics/PorterDuffColorFilter.html)

  ``` java
  PorterDuffColorFilter (int color, PorterDuff.Mode mode)
  ```

* [Paint](https://developer.android.com/reference/android/graphics/Paint.htm)

  ``` java
  Xfermode setXfermode (Xfermode xfermode)
  ```

  >Xfermode 注意事项：
  >
  >1. 使用离屏缓冲（Off-screen Buffer）
  >
  >   使用 `Canvas.saveLayout()` 可以做短时的离屏缓冲，在绘制之前调用保存，绘制之后恢复：
  >
  >   ``` java
  >   int saved = canvas.saveLayer(null, null, Canvas.ALL_SAVE_FLAG);
  >   canvas.drawBitmap(rectBitmap, 0, 0, paint); // 画方
  >   paint.setXfermode(xfermode); // 设置 Xfermode
  >   canvas.drawBitmap(circleBitmap, 0, 0, paint); // 画圆
  >   paint.setXfermode(null); // 用完及时清除 Xfermode
  >   canvas.restoreToCount(saved);
  >   ```
  >
  >   还可以使用 `View.setLayerType()` 直接将整个 View 都绘制在离屏缓冲中。
  >
  >2. 控制好透明区域
  >
  >   要让它足够覆盖到要和它结合绘制的内容。
  >
  >   ![透明区域](https://ws1.sinaimg.cn/large/006tNc79ly1fig73037soj30sj0x3myt.jpg)