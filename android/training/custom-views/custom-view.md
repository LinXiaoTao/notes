[参考](https://developer.android.com/training/custom-views/create-view.html)

### 创建View类

一个自定义 View 应该：

* 符合 Android 规范
* 提供适用于 Android XML 布局的自定义样式属性
* 发送辅助事件
* 兼容多个Android 平台

#### View的子类

所有在 Android 框架中定义的视图类都扩展了View。您的自定义视图也可以直接扩展视图，也可以通过扩展其中一个现有视图子类来节省时间。

要让Android Studio与您的视图进行交互，至少必须提供一个构造函数，它将 `Context`和 `AttributeSet` 对象作为参数。

```java
class PieChart extends View {
    public PieChart(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```

#### 定义自定义属性

要在 XML 中使用自定义属性，你必须：

* 在<declare-styleable>资源元素中为视图定义自定义属性
* 指定XML布局中属性的值
* 在运行时检索属性值
* 将检索的属性值应用于您的视图

要定义自定义属性，请将 <declare-styleable> 资源添加到项目中。通常将这些资源放入 `res/values/attrs.xml` 文件中。

```xml
<resources>
   <declare-styleable name="PieChart">
       <attr name="showText" format="boolean" />
       <attr name="labelPosition" format="enum">
           <enum name="left" value="0"/>
           <enum name="right" value="1"/>
       </attr>
   </declare-styleable>
</resources>
```

styleable 的名称与自定义 View 的名称一般相同。

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">
 <com.example.customviews.charting.PieChart
     custom:showText="true"
     custom:labelPosition="left" />
</LinearLayout>
```

注意将自定义视图添加到布局的XML标签的名称。它是自定义视图类的完全限定名称。如果您的视图类是内部类，则必须使用视图的外部类的名称进一步限定它。例如：`com.example.customviews.charting.PieChart$PieView`。

#### 应用自定义属性

虽然可以直接从 `AttributeSet` 读取值，但这样做有一些缺点：

* 属性值中的资源引用未解决
* 样式不适用

对于 res 目录中的每个 `<declare-styleable>` 资源，生成的 `R.java` 定义了一个属性 id 数组和一组常量，用于定义数组中每个属性的索引。您可以使用预定义的常量从TypedArray读取属性。

```java
public PieChart(Context context, AttributeSet attrs) {
   super(context, attrs);
   TypedArray a = context.getTheme().obtainStyledAttributes(
        attrs,
        R.styleable.PieChart,
        0, 0);

   try {
       mShowText = a.getBoolean(R.styleable.PieChart_showText, false);
       mTextPos = a.getInteger(R.styleable.PieChart_labelPosition, 0);
   } finally {
       a.recycle();
   }
}
```

请注意，TypedArray 对象是一个共享资源，必须在使用后被回收。

#### 添加属性和事件

属性是控制视图的行为和外观的有力方式，但是只有在视图初始化时才能读取它们。为了提供动态行为，为每个自定义属性公开一个属性 getter 和 setter。

注意必要时候，需要调用 `invalidate()` 和 `requestLayout()` ，这些调用对于确保视图的可行性至关重要。

自定义视图还应支持事件侦听器来传达重要事件。

#### 无障碍设计

您的自定义视图应该支持最广泛的用户。这包括残疾用户，阻止他们看到或使用触摸屏。为了支持残疾用户，您应该：

* 使用 `android:contentDescription` 属性标记输入字段
* 通过在适当时调用sendAccessibilityEvent（）来发送辅助功能事件。
* 支持备用控制器，如 D-pad 和轨迹球

### 自定义绘制

自定义视图的最重要部分是它的外观。

#### 覆盖onDraw()

绘制自定义视图最重要的步骤是覆盖 `onDraw()` 方法。

#### 创建绘制对象

`android.graphics` 框架将绘制分为两个部分：

* 通过 `Canvas` 处理绘制什么
* 通过 `Paint` 处理怎么绘制

提前创建对象是一个重要的优化。视图被非常频繁地重绘，并且许多绘图对象需要昂贵的初始化。在您的 `onDraw()` 方法中创建绘图对象可以显着降低性能，并可能使您的 UI 看起来迟钝。

#### 处理布局事件

虽然View具有许多处理测量的方法，但大多数方法不需要被覆盖。如果您的视图不需要对其大小进行特殊控制，则只需覆盖一个方法：`onSizeChanged()` 。当您的视图首次分配大小时，会调用 `onSizeChanged()`  ，如果视图的大小由于任何原因而更改，则再次调用。

当您的视图被分配一个大小时，布局管理器假定大小包括所有视图的填充。计算视图大小时，必须处理填充值( padding )。

如果您需要对视图的布局参数进行更好的控制，请执行 `onMeasure()` 。这个方法的参数是 `View.MeasureSpec` 值，它告诉你你的视图的父母想要你的视图有多大，以及这个大小是最大的还是只是一个建议。

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
   // 尝试根据最小值来计算宽度
   int minw = getPaddingLeft() + getPaddingRight() + getSuggestedMinimumWidth();
   int w = resolveSizeAndState(minw, widthMeasureSpec, 1);

   // 不管宽度如何，尽可能要最大值
   int minh = MeasureSpec.getSize(w) - (int)mTextWidth + getPaddingBottom() + getPaddingTop();
   int h = resolveSizeAndState(MeasureSpec.getSize(w) - (int)mTextWidth, heightMeasureSpec, 0);

   setMeasuredDimension(w, h);
}
```

在这段代码中有三个重要的事项要注意：

* 计算考虑了视图的填充。
* 使用帮助方法 `resolveSizeAndState()` 用于创建最终的宽度和高度。
* `onMeasure()` 没有返回值。相反，该方法通过调用 `setMeasureDimension()` 来传递其结果。调用这个方法时必须的。否则，将抛出运行时异常。

#### 绘制

一旦定义了对象创建和测量代码，就可以实现 `onDraw()` 。每个视图都以不同的方式实现 `onDraw()` ，但是大多数视图共享一些常见的操作：

* 使用 `drawText()` 绘制文本。通过调用 `setTypeface()` 指定字体，并通过调用 `setColor()` 指定文本颜色。
* 使用 `drawRect()` ，`drawOval()` 和 `drawArc()` 绘制原始形状。通过调用 `setStyle()` 来更改形状是否填充，勾勒或者两者。
* 使用 `Path` 类绘制更复杂的形状。通过将线条和曲线添加到 `Path` 对象来定义形状，然后通过使用 `drawPath()` 绘制形状。与原始形状一样，`Path` 可以被勾勒，填充或两者都取决于 `setStyle()` 。
* 通过创建 `LinearGradient` 对象来定义渐变填充。调用 `setShader()` 在填充的形状上使用 `LinearGradient`。
* 使用 `drawBitmap()` 绘制位图。 

### 与视图交互

绘制UI只是创建自定义视图的一部分。您还需要使您的视图以与您模拟的真实世界行为非常相似的方式对用户输入进行响应。

#### 处理手势

可以覆盖回调以自定义应用程序对用户的响应。 Android系统中最常见的输入事件是 touch ，触发`onTouchEvent(android.view.MotionEvent)`。覆盖此方法来处理事件：

```java
@Override
   public boolean onTouchEvent(MotionEvent event) {
    return super.onTouchEvent(event);
   }
```

要将原始触摸事件转换为手势，Android 提供 `GestureDetector` 。

#### 创建合理的物理运动

幸运的是，Android  提供帮助类来模拟这个和其他行为。 `Scroller` 类是处理飞轮式 `fling` 手势的基础。

要开始 `fling` ，请以起始速度和 `fling` 的最小和最大 x 和 y 值调用 `fling()` 。对于速度值，您可以使用 `GestureDetector` 为您计算的值。

> 注意：虽然 `GestureDetector` 计算出的速度在物理上是准确的，但许多开发人员认为使用这个值可以使得动画动画变得太快。将 x 和 y 速度除以 4 到 8 是很常见的。

调用 `fling()` 时会为手势设置物理模型，然后，通过定期调用 `Scroller.computeScrollOffset()` 来更新 `Scroller` 。`computeScrollOffset()` 通过读取当前时间和使用物理模型来计算 x 和 y 的位置更新 `Scroller` 对象的内部状态。调用 `getCurrX` 和 `getCurrY` 来获取这些值。

大多数 `View` 通过将 `Scroller` 对象的 x,y 传递到 `scrollTo` 。

`Scroller` 类为您计算滚动位置，但不会自动将这些位置应用于您的视图。您有责任确保您获得并应用新坐标，足以使滚动动画看起来顺利。有两种方法可以做到这一点：

* 在调用 `fling()` 之后调用 `postInvalidate()` ，以命令强制重绘。这种技术要求你在每次滚动偏移量更改时，在 `onDraw()` 中计算滚动偏移量，同时调用 `postInvalidate()` 。
* 使用 `ValueAnimator` 在动画的期间 `fling` ，并通过调用 `addUpdateListener()` 添加一个监听器来处理动画更新。

> 注意：可以在较低 API 级别的应用程序中使用 `ValueAnimator` 。只需确保在运行时检查当前 API 级别，如果小于 11，则忽略对视图动画系统的调用。

#### 使过渡平滑

Android 从 3.0 开始提供 `property animation framework` ，用来使平滑过渡更加容易。

### 优化自定义View

为了避免在播放过程中感觉迟钝或短暂的UI，请确保动画始终以每秒 60 帧的速度运行。

#### Do Less,Less Frequently

为了加速你的 view，对于频繁调用的方法，需要尽量减少不必要的代码。先从 `onDraw` 开始，需要特别注意不应该在这里做内存分配的事情，因为它会导致 GC，从而导致卡顿。在初始化或者动画间隙期间做分配内存的动作。不要在动画正在执行的时候做内存分配的事情。

你还需要尽可能的减少 `onDraw` 被调用的次数，大多数时候导致onDraw都是因为调用了invalidate().因此请尽量减少调用invaildate()的次数。

另一个非常昂贵的操作是遍历布局。任何时候，一个视图调用 `requestLayout()`，Android UI 系统需要遍历整个视图层次结构，以了解每个视图需要多大。如果发现冲突的测量，则可能需要多次遍历层次结构。这些深层次的层次结构导致性能问题。使您的视图层次结构尽可能浅。

如果您有一个复杂的UI，请考虑编写一个自定义 `ViewGroup` 来执行其布局。与内置视图不同，您的自定义视图可以针对其子项的大小和形状进行特定于应用程序的假设，从而避免遍历其子项以计算度量。