[参考](https://developer.android.com/training/custom-views/making-interactive.html?hl=zh-cn)

# 与视图交互

绘制 UI 只是创建自定义视图的一部分。您还需要使您的视图以与您模拟的真实世界行为非常相似的方式对用户输入进行响应。



## 处理手势

可以覆盖回调以自定义应用程序对用户的响应。 Android 系统中最常见的输入事件是 **touch** ，触发`onTouchEvent(android.view.MotionEvent)`。覆盖此方法来处理事件：

``` java
   @Override
   public boolean onTouchEvent(MotionEvent event) {
    return super.onTouchEvent(event);
   }
```

要将原始触摸事件转换为手势，Android 提供  `GestureDetector` 。



## 创建合理的物理运动

手势是控制触摸屏幕设备的直接方式，但是它们可能是有违常理，而且难以记住，除非它们产生合理的物理运动。一个很好的例子就是 **fling** 手势，当用户快速滑动屏幕，然后抬起手指。如果 UI 通过快速向 **fling** 的方向移动，然后放慢速度来响应，那么这个手势就是有意义的。

Android 提供了 [Scroller](https://developer.android.com/reference/android/widget/Scroller.html) 类来处理这种 **fling** 手势。

要开始一个 **fling** 处理，通过传递开始速度、最小和最大的 x，y 值来调用 `fiing()` 方法。可以通过使用 `GestureDetector` 来获取速度。

``` java
@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
   mScroller.fling(currentX, currentY, velocityX / SCALE, velocityY / SCALE, minX, minY, maxX, maxY);
   postInvalidate();
}
```

> 注意：虽然  `GestureDetector`  计算出的速度在物理上是准确的，但许多开发人员认为使用这个值可以使得动画动画变得太快。将 x 和 y 速度除以 4 到 8 是很常见的。

调用 `fling()` 时会为手势设置物理模型，然后，通过定期调用 `Scroller.computeScrollOffset()` 来更新 `Scroller` 。`computeScrollOffset()` 通过读取当前时间和使用物理模型来计算 x 和 y 的位置更新 `Scroller` 对象的内部状态。调用  `getCurrX()`  和  `getCurrY()` 来获取这些值。

大多数 `View` 通过将 `Scroller` 对象的 x,y 传递到 `scrollTo()` 。

`Scroller` 类为您计算滚动位置，但不会自动将这些位置应用于您的视图。您有责任确保您获得并应用新坐标，足以使滚动动画看起来顺利。有两种方法可以做到这一点：

- 在调用 `fling()` 之后调用 `postInvalidate()` ，以命令强制重绘。这种技术要求你在每次滚动偏移量更改时，在 `onDraw()` 中计算滚动偏移量，同时调用 `postInvalidate()` 。
- 使用 `ValueAnimator` 在动画的期间 `fling` ，并通过调用 `addUpdateListener()` 添加一个监听器来处理动画更新。

> 注意：可以在较低 API 级别的应用程序中使用 `ValueAnimator` 。只需确保在运行时检查当前 API 级别，如果小于 11，则忽略对视图动画系统的调用。

``` java
	   mScroller = new Scroller(getContext(), null, true);
       mScrollAnimator = ValueAnimator.ofFloat(0,1);
       mScrollAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
           @Override
           public void onAnimationUpdate(ValueAnimator valueAnimator) {
               if (!mScroller.isFinished()) {
                   mScroller.computeScrollOffset();
                   setPieRotation(mScroller.getCurrY());
               } else {
                   mScrollAnimator.cancel();
                   onScrollFinished();
               }
           }
       });
```



## 使过渡平滑

在 Android 3.0 中引入的 [Android 属性动画](https://developer.android.com/guide/topics/graphics/prop-animation.html) 使平滑过渡变得容易。

只要属性更改会影响视图的外观，就可以使用属性动画，不要直接更改属性。

``` java
mAutoCenterAnimator = ObjectAnimator.ofInt(PieChart.this, "PieRotation", 0);
mAutoCenterAnimator.setIntValues(targetAngle);
mAutoCenterAnimator.setDuration(AUTOCENTER_ANIM_DURATION);
mAutoCenterAnimator.start();
```

