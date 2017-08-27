[参考](This lesson teaches you run an animation between two scenes using built-in transitions to move, resize, and fade views. The next lesson shows you how to define custom transitions.)

# 应用 Transition

在框架中，动画创建一系列的帧，描述了起始和结束 scenes 中的视图层次结构之间的变化。框架将这些动画表示为 transition 对象，其中包含有关动画的信息。要运行动画，您可以向 transition manager 提供使用的 transition 和 结束 scene。



## 创建 Transition

内置的 transition 类型。

| 类                                        | 标签                | 属性                                       | 效果                                       |
| ---------------------------------------- | ----------------- | ---------------------------------------- | ---------------------------------------- |
| [AutoTransition](https://developer.android.com/reference/android/transition/AutoTransition.html) | <autoTransition/> | -                                        | 默认，淡出，移动和调整大小，并按照顺序淡出视图。                 |
| [Fade](https://developer.android.com/reference/android/transition/Fade.html) | <fade/>           | android:fadingMode="[fade_in \|fade_out \|fade_in_out]" | **fade_in_out**（默认）执行 **fade_out**，然后是 **fade_in** |
| [ChangeBounds](https://developer.android.com/reference/android/transition/ChangeBounds.html) | <changeBounds/>   | -                                        | 视图移动并且调整大小                               |



### 从资源文件中创建 transition 实例

`res/transition/fade_transition.xml`

``` xml
<fade xmlns:android="http://schemas.android.com/apk/res/android" />
```

``` java
Transition mFadeTransition =
        TransitionInflater.from(this).
        inflateTransition(R.transition.fade_transition);
```



### 在您的代码中创建 transition 实例

``` java
Transition mFadeTransition = new Fade();
```



## 应用 Transition

``` java
TransitionManager.go(mEndingScene, mFadeTransition);
```

在运行 transition 实例指定的动画效果过程中，框架将根据结束 scene 中的层次视图结构来改变当前 scene root 中的视图层次结构。起始 scene 是上一次 transition 的结束 scene。如果之前没有 transition，则从用户界面的当前状态自动确定起始 scene。

如果不指定 transition 实例，则 transition manager 可以应用自动 transition，这对大多数情况都是合理的。



## 选择特定的目标视图

默认情况下，框架将 transition 应用于开始和结束 scene 中的所有视图。在某些情况下，您可能只想将动画应用于场景中的一部分视图。

transition 动画的每个视图称为目标视图。您只能选择与 scene 相关联的视图层次结构的一部分视图，作为目标视图。

要从目标列表中删除一个或多个视图，请在开始 transition 之前调用 `removeTarget()` 方法。要仅将您指定的视图添加到目标列表，请调用 `addTarget()` 方法。使用 `addTarget()` 后，transition 只能使用这些目标视图。 



## 指定多个 Transitions

要从 XML 中的 transition 集合中定义 transition set，请在 `xml/transitions` 目录中创建资源文件，并列出 `<transitionSet/>` 元素下的 transition。

``` xml
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android"
    android:transitionOrdering="sequential">
    <fade android:fadingMode="fade_out" />
    <changeBounds />
    <fade android:fadingMode="fade_in" />
</transitionSet>
```

在代码中调用 `TransitionInflater.from()` 方法来获取 `TransitionInflater` 实例来 inflate transition set 到 `TransitionSet` 对象中。`TransitionSet` 类是继承于 `Transition` 类。



## 使用没有 Scenes 的 Transition

更改视图层次结构不是修改用户界面的唯一方法。您还可以通过在当前层次结构中添加，修改和删除子视图进行更改。

如果使用这种方式在当前视图层次结构中进行更改，则无需创建 Scene。相反，在同个视图层次结构的两个状态之间，创建并应用一个延迟的 transition。框架以当前视图层次结构为开始，记录对视图所做的更改，并在系统重新绘制用户界面时应用 transition 的动画效果。



`activity_main.xml`

``` xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/mainLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <EditText
        android:id="@+id/inputText"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    ...
</RelativeLayout>
```



`MainActivity.java`

``` java
private TextView mLabelText;
private Fade mFade;
private ViewGroup mRootView;
...

// Load the layout
this.setContentView(R.layout.activity_main);
...

// Create a new TextView and set some View properties
mLabelText = new TextView();
mLabelText.setText("Label").setId("1");

// Get the root view and create a transition
mRootView = (ViewGroup) findViewById(R.id.mainLayout);
mFade = new Fade(IN);

// Start recording changes to the view hierarchy
TransitionManager.beginDelayedTransition(mRootView, mFade);

// Add the new TextView to the view hierarchy
mRootView.addView(mLabelText);

// When the system redraws the screen to show this update,
// the framework will animate the addition as a fade in
```



## 定义 Transition 的生命周期回调

transition 生命周期表示框架在调用 `TransitionManager.go()` 方法和完成动画之间的时间期间的 transition 状态。在重要的生命周期状态下，框架调用由 `transitionListener` 接口定义的回调。

transition 生命周期对于在 scene 更换期间，将视图属性值从起始视图层次结构复制到结束视图层次结构非常有用。您不能将属性值简单地从起始视图复制到结束视图中，因为结束视图层次结构在 transition 结束之前不会被 inflate。相反，您需要将值存储，然后在框架完成 transition 后，将值复制到结束视图层次结构中。

