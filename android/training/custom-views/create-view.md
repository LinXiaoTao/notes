[参考](https://developer.android.com/training/custom-views/create-view.html?hl=zh-cn)

# 创建视图类

一个自定义视图应该：

- 符合 Android 规范
- 提供适用于 Android XML 布局的自定义样式属性
- 发送辅助事件
- 兼容多个 Android 平台



## 视图的子类

所有在 Android 框架中定义的视图类都扩展了 `View`。您的自定义视图也可以直接扩展视图，也可以通过扩展其中一个现有视图来节省时间。

要让 Android Studio 与您的视图进行交互，至少必须提供一个构造函数，它将 `Context`  和  `AttributeSet` 对象作为参数。这个构造方法允许布局编辑器创建和编辑您的视图实例。

``` java
class PieChart extends View {
    public PieChart(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```



## 定义自定义属性

要在 XML 中使用自定义属性，你必须：

- 在 `<declare-styleable>` 资源元素中为视图定义自定义属性
- 指定 XML 布局中属性的值
- 在运行时检索属性值
- 将检索的属性值应用于您的视图

要定义自定义属性，请将 `<declare-styleable>` 资源添加到项目中。通常将这些资源放入 `res/values/attrs.xml`  文件中。

``` xml
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

一般情况下，`declare-styleable` 的名称和自定义视图的名称一样。

一旦定义了自定义属性，就可以像内置的属性一样在布局 XML 文件中使用它们。唯一的区别是，您的自定义属性属于不同的命名空间。而不是属于 `http://schemas.android.com/apk/res/android`  命名空间，它们属于 `http://schemas.android.com/apk/res/[your package name]` ，比如，这里是演示如何使用 `PieChart` 定义的属性：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">
 <com.example.customviews.charting.PieChart
     custom:showText="true"
     custom:labelPosition="left" />
</LinearLayout>
```

为了避免使用太长的命名空间 URI，这个例子使用 `xmlns` 指令，这个指令将 `custom` 设置为命名空间 `http://schemas.android.com/apk/res/com.example.customviews` 的别名。

注意将自定义视图添加到布局的 XML 标签的名称。它是自定义视图类的完全限定名称。如果您的视图是内部类，您必须使用视图的外部类的名称进一步限定它。例如：`com.example.customviews.charting.PieChart$PieView`。



## 应用自定义属性

当从 XML 布局创建视图时，XML 标签中的所有属性都将从资源包中读取，并作为一个 [AttributeSet](https://developer.android.com/reference/android/util/AttributeSet.html?hl=zh-cn) 传递给视图的构造方法。虽然可以直接从  `AttributeSet`  读取值，但这样做有一些缺点：

- 属性值中的资源引用未解决
- 样式不适用

相反，将这个 `AttributeSet` 传递给 [obtainStyledAttributes()](https://developer.android.com/reference/android/content/res/Resources.Theme.html?hl=zh-cn#obtainStyledAttributes(android.util.AttributeSet, int[], int, int))。这个方法将返回一个已经取消引用和应用样式的 [TypeArray](https://developer.android.com/reference/android/content/res/TypedArray.html?hl=zh-cn) 数组。

对于 res 目录中的每个  `<declare-styleable>` 资源，生成的 `R.java` 定义了一个属性 id 数组和一组常量，用于定义数组中每个属性的索引。您可以使用预定义的常量从TypedArray读取属性。

``` java
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

`TypedArray` 是一个共享资源，记得使用后进行回收。



## 添加属性和事件

属性是控制视图的行为和外观的有力方式，但是只有在视图初始化时才能读取它们。为了提供动态行为，为每个自定义属性公开一个属性 getter 和 setter。

注意必要时候，需要调用  `invalidate()`  和  `requestLayout()` ，这些调用对于确保视图的可行性至关重要。

自定义视图还应支持事件侦听器来传达重要事件。



## 无障碍设计

您的自定义视图应该支持最广泛的用户。这包括残疾用户，阻止他们看到或使用触摸屏。为了支持残疾用户，您应该：

- 使用  `android:contentDescription` 属性标记输入字段
- 通过在适当时调用 `sendAccessibilityEvent()` 来发送辅助功能事件。
- 支持备用控制器，如 D-pad 和轨迹球