[参考](https://developer.android.com/training/custom-views/custom-drawing.html?hl=zh-cn)

# 自定义绘制

自定义视图的最重要部分是它的外观。



## 覆盖 onDraw()

绘制自定义视图最重要的步骤是覆盖 `onDraw()` 方法。`onDraw()` 的参数是 `Canvas` 对象，可以用来绘制自身。`Canvas` 类定义了绘制文本、行、位图和许多其他图形。您可以在 `onDraw()` 方法中使用 `Canvas` 的方法来绘制您的自定义 UI。

在您调用任何绘图方法之前，必须创建一个 `Paint` 对象。



## 创建绘制对象

[android.graphics](https://developer.android.com/reference/android/graphics/package-summary.html?hl=zh-cn) 框架将绘制分为两个部分：

- 通过 `Canvas` 处理绘制什么
- 通过 `Paint` 处理怎么绘制

简单的说，`Canvas` 可以定义您可以在屏幕上绘制的形状，而 `Paint` 定义您绘制的每个形状的颜色、样式、字体等等。

``` java
private void init() {
   mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mTextPaint.setColor(mTextColor);
   if (mTextHeight == 0) {
       mTextHeight = mTextPaint.getTextSize();
   } else {
       mTextPaint.setTextSize(mTextHeight);
   }

   mPiePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mPiePaint.setStyle(Paint.Style.FILL);
   mPiePaint.setTextSize(mTextHeight);

   mShadowPaint = new Paint(0);
   mShadowPaint.setColor(0xff101010);
   mShadowPaint.setMaskFilter(new BlurMaskFilter(8, BlurMaskFilter.Blur.NORMAL));

   ...
```

提前创建对象是一个重要的优化。视图被非常频繁地重绘，并且许多绘图对象需要昂贵的初始化。在您的 `onDraw()` 方法中创建绘图对象可以显着降低性能，并可能使您的 UI 看起来迟钝。



## 处理布局事件

虽然 `View` 具有许多处理测量的方法，但大多数方法不需要被覆盖。如果您的视图不需要对其大小进行特殊控制，则只需覆盖一个方法：`onSizeChanged()` 。当您的视图首次分配大小时，会调用 `onSizeChanged()`  ，如果视图的大小由于任何原因而更改，则再次调用。

当您的视图被分配一个大小时，布局管理器假定大小包括所有视图的填充。计算视图大小时，必须处理填充值( padding )。

``` java
	   // Account for padding
       float xpad = (float)(getPaddingLeft() + getPaddingRight());
       float ypad = (float)(getPaddingTop() + getPaddingBottom());

       // Account for the label
       if (mShowText) xpad += mTextWidth;

       float ww = (float)w - xpad;
       float hh = (float)h - ypad;

       // Figure out how big we can make the pie.
       float diameter = Math.min(ww, hh);
```

如果您需要对视图的布局参数进行更好的控制，请执行  `onMeasure()` 。这个方法的参数是  `View.MeasureSpec` 值，它告诉你你的视图的父母想要你的视图有多大，以及这个大小是最大的还是只是一个建议。

``` java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
   // Try for a width based on our minimum
   int minw = getPaddingLeft() + getPaddingRight() + getSuggestedMinimumWidth();
   int w = resolveSizeAndState(minw, widthMeasureSpec, 1);

   // Whatever the width ends up being, ask for a height that would let the pie
   // get as big as it can
   int minh = MeasureSpec.getSize(w) - (int)mTextWidth + getPaddingBottom() + getPaddingTop();
   int h = resolveSizeAndState(MeasureSpec.getSize(w) - (int)mTextWidth, heightMeasureSpec, 0);

   setMeasuredDimension(w, h);
}
```

在这段代码中有三个重要的事项要注意：

- 计算考虑了视图的填充。
- 使用帮助方法 `resolveSizeAndState()` 用于创建最终的宽度和高度。
- `onMeasure()` 没有返回值。相反，该方法通过调用 `setMeasureDimension()` 来传递其结果。调用这个方法时必须的。否则，将抛出运行时异常。



## 绘制！

一旦定义了对象创建和测量代码，就可以实现 `onDraw()` 。每个视图都以不同的方式实现 `onDraw()` ，但是大多数视图共享一些常见的操作：

- 使用  `drawText()` 绘制文本。通过调用  `setTypeface()` 指定字体，并通过调用 `setColor()` 指定文本颜色。
- 使用  `drawRect()` ，`drawOval()` 和 `drawArc()` 绘制原始形状。通过调用 `setStyle()` 来更改形状是否填充，勾勒或者两者。
- 使用  `Path` 类绘制更复杂的形状。通过将线条和曲线添加到 `Path` 对象来定义形状，然后通过使用 `drawPath()` 绘制形状。与原始形状一样，`Path` 可以被勾勒，填充或两者都取决于 `setStyle()` 。
- 通过创建  `LinearGradient`  对象来定义渐变填充。调用 `setShader()` 在填充的形状上使用 `LinearGradient`。
- 使用 `drawBitmap()` 绘制位图。 