[参考](http://hencoder.com/ui-1-3/)



使用 `Canvas.drawText()` 的一些注意事项：

* `Canvas.drawText(String text,float x,float y,Paint paint)` 方法中，y 值指的是文字的  **基线（baseline）**。

* `Paint.getFontMetrics(FontMetrics)` 方法根据 Paint 的字体和字号，返回字体的推荐行间距。FontMetrics 提供了文字排印的数值：`ascent`, `descent`, `top`, `bottom`, `leading`。

  ![FontMetrics](https://ws3.sinaimg.cn/large/52eb2279ly1fig66iud3gj20ik0bn41l.jpg)

  * baseline：文字显示的基准线
  * ascent / descent：限制普通字符的顶部和底部范围。
  * top / bottom：限制所有字形（glyph）的顶部和底部范围。
  * leading：行的额外间距，即对于上下相邻的两行，上行的 bottom 线和下行的 top 线的距离。

* `Paint.getTextBounds(String,int,int,Rect)` 获取文字的显示范围。

  这里需要注意的是：Rect 返回的值是 top，bottom 的值是基于 baseline 的，位于 baseline 之上为正，之下为负数。

* `Paint.measureText(String,int,int)` 测量文字的宽度并返回。

  一般来说，`measureText()` 测量的宽度总是要比 `getTextBounds()` 要大一点，

  * `getTextBounds()`：它测量的是文字的显示范围（关键词：显示）。形象点来说，你这段文字外放置一个可变的矩形，然后把矩形尽可能地缩小，一直小到这个矩形恰好紧紧包裹住文字，那么这个矩形的范围，就是这段文字的 bounds。
  * `measureText()`：它测量的是文字绘制时所占用的宽度（关键词：占用）。前面已经讲过，一个文字在界面中，往往需要占用比他的实际显示宽度更多一点的宽度，以此来让文字和文字之间保留一些间距，不会显得过于拥挤。

