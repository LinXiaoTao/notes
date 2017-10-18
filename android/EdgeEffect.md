## 简介

当用户滚动超过 2D 空间的内容边界时，此类执行在可滚动控件的边缘处使用的图形效果。

EdgeEffect 是有状态的。使用 EdgeEffect 的自定义控件应该为应显示效果的每个边缘创建一个实例，使用 `onAbsorb(int)`，`onPull(float)` 和 `onRelease()` 方法提供输入数据，并使用 `darw(Canvas` 绘制效果。如果 `isFinished()` 在绘制后返回 `false`，则应该在下一帧继续绘制。

自定义控件绘制时，应先绘制其主要内容和子视图，通常通过覆盖 `View.draw()` 方法来调用 EdgeEffect 的 `draw(Canvas)` 方法来将边缘效果绘制在视图内容的顶部。



### API

* `onPull(float deltaDistance,float displacement)`

  当用户拖动边缘效果时调用，这将更新当前视觉效果及其相关动画的状态。视图应该在此之后调用 `invalidate()` ，来调用视图重新绘制。

  `deltaDistance`：自上次调用后的距离更改，值可以是 0（不改变） 到 1.f（EdgeEffect Height）或负值。

  `displacement`：开始拉动的点距离起始侧的距离，值可以是 0 到 1。

* `onAbsorb(int velocity)`

  当效果是以给定速度拖动时调用，当使用 **fling** 手势的时候使用。

  `velocity`: **fling** 时候的速度 

* `draw(Canvas canvas)`

  将效果绘制在画布上。

  > 绘制效果将从 x=0 到 x=width 进行全绘制，并从 y=0 开始绘制到对应的 y 值。



### 重要

从 `draw(Canvas)` 方法我们可以知道，EdgeEffect 是没有绘制方向的，它只会在 Y 轴上绘制对应的拖动效果。这也就需要我们在调用 EdgeEffect 绘制时，去操作 Canvas，从而达到想要的效果。



### 使用

我们从 android-25 的 `HorizontalScrollView` 来学习 EdgeEffect 的使用。

``` java
//定义和构造 EdgeEffect
private EdgeEffect mEdgeGlowLeft;
private EdgeEffect mEdgeGlowRight;

Context context = getContext();
mEdgeGlowLeft = new EdgeEffect(context);
mEdgeGlowRight = new EdgeEffect(context);
```

``` java
public boolean onTouchEvent(MotionEvent ev){
  //省略
  if (canOverscroll) {
    final int pulledToX = oldX + deltaX;
    if (pulledToX < 0) {
      //当滑动到了边缘后，调用 onPull
      mEdgeGlowLeft.onPull((float) deltaX / getWidth(),
             1.f - ev.getY(activePointerIndex) / getHeight());
      if (!mEdgeGlowRight.isFinished()) {
        mEdgeGlowRight.onRelease();
        }
      //省略
}
```

``` java
public void computeScroll(){
  if (mScroller.computeScrollOffset()) {
  	//省略
    if (canOverscroll) {
    	if (x < 0 && oldX >= 0) {
          //当 fling 手势滚动到边缘后，调用 onAbsorb，注意不要多次调用 onAbsorb
           mEdgeGlowLeft.onAbsorb((int) mScroller.getCurrVelocity());
         }
      //省略
    }
  }	
}
```

``` java
//需要先调用 super.draw(Canvas)，这样才能将边缘效果绘制在内容视图之上
public void draw(Canvas canvas) {
  super.draw(canvas);
  if (mEdgeGlowLeft != null) {
    final int scrollX = mScrollX;
    if (!mEdgeGlowLeft.isFinished()) {
    	final int restoreCount = canvas.save();
      	final int height = getHeight() - mPaddingTop - mPaddingBottom;
      	
      	//这里需要对 canvas 进行旋转和位移
      	canvas.rotate(270);
      	canvas.translate(-height + mPaddingTop, Math.min(0, scrollX));
      	mEdgeGlowLeft.setSize(height, getWidth());
      	if (mEdgeGlowLeft.draw(canvas)) {
        	postInvalidateOnAnimation();
        }
      	canvas.restoreToCount(restoreCount);
      	//省略
    }
}
```

![img](http://oy017242u.bkt.clouddn.com/EdgeEffects.png)

上图中，X/Y 是原来的坐标系，X'/Y' 是旋转后的坐标系，矩形表示边缘效果，因为 EdgeEffect 是从 X=0 到 X=width 进行全绘制，所以需要将坐标系进行旋转，即 `canvas.rotate(270);` 来达到我们需要的效果。 当坐标系旋转后，需要将边缘效果按新的坐标系的 X' 向负方向移动，即 `canvas.translate(-height + mPaddingTop, Math.min(0, scrollX));` ，这样才能处于可见范围内。