[参考](https://developer.android.com/training/transitions/overview.html)

# Transition 框架

动画给您的应用的用户界面提供视觉吸引力。动画突出显示改变，同时帮助用户了解您的应用的工作原理。

为了帮助您在一个视图层次结构和另一个层次结构之间进行动画改变，Android 提供了 Transition 框架。这个框架在同个层次结构中的所有视图改变时，为这些视图应用一个或多个动画。

这个框架有以下功能：

* 组级的动画

  在同个层次结构中所有视图上，应用一个或多个动画

* 基于过渡的动画

  基于视图的起始属性值和结束属性值之间运行动画

* 内置动画

  包含预先定义常见效果的动画，比如淡入淡出或者移动

* 支持资源文件

  从布局资源文件中加载视图层次结构和内置动画

* 生命周期回调

  定义回调来提供更好地控制动画和层次结构改变处理



## 概括

框架将动画运用到两个视图层次结构的所有的视图。一个视图层次机构可以是只包含单个视图那样简单，也可以像 `ViewGoup` 那样包含一个精心设计的视图树。框架通过在初始（开始）视图层次结构与最终（结束）视图层次结构之间随时间改变其中一个或多个属性值，来对每个视图应用动画效果。

过渡框架与视图层次结构和动画并行。框架的目的是存储视图层次结构，在这些层次结构之间进行更改，以便修改设备屏幕的外观，并通过存储和应用动画定义来让改变产生动画效果。

视图层次结构，框架对象和动画之间的关系：

![过渡框架中的关系](https://developer.android.com/images/transitions/transitions_diagram.png)

过渡框架抽象出 Scenes，Transition 和 Transition Manager。要使用框架，您应该为应用中的视图层次结构创建计划在视图层次结构之间改变的 Scenes。下一步，可以为所有您想要使用的动画创建一个 Transition。要在两个视图层次结构之间启动动画，请使用一个 Transition Manager 来指定要使用的 Transition 和 结束 Scenes。



## 场景（Scenes）

一个 Scenes 保存当前视图层次结构的状态，包括它的所有视图和视图对应的属性。保存视图层次结构的状态可以让您从一个 Scenes 过渡到另外一个 Scenes 的状态。

框架让您可以从布局资源文件中创建 Scenes ，或者在您的代码中从 `ViewGroup` 实例中创建 Scenes 。

大部分情况下，您并不需要去创建一个起始的 Scenes 。如果您去应用一个 Transition，框架会使用上一个结束场景来作为任何后续过渡的起始场景。如果您没有应用一个过渡效果，框架会从当前场景状态中收集视图的有关信息。

当你做一个 Scenes 变化时，您可以在 Scenes 中定义运行时它自己的动作。比如，在您过渡完一个 Scenes 后去清理视图的设置。

 Scenes 还存储对视图层次结构的父级的引用。这个根视图被称为 **scene root**。改变 Scenes 和影响 Scenes 的动画都发生在 **scene root** 里面。



## 过渡效果（Transitions）

在框架中，有关动画的信息存储在 [Transition](https://developer.android.com/reference/android/transition/Transition.html) 对象中。要运行动画，请使用一个 Transition Manager 的实例来应用 Transition。框架可以在不同的场景之间过渡，或者转换到当前场景的不同状态。

框架已经包括一组内置的 Transition，用于常用的动画效果。您也可以使用动画框架中的 API 定义自定义的  Transition 来创建动画效果。

Transition 的生命周期类似 Activity 的生命周期，它代表框架在动画开始和完成之间监视的 Transition 的状态。在重要的生命周期状态下，框架调用回调方法，您可以实现这些方法，以便在 Transition 的不同阶段对用户界面进行调整。



## 限制

框架有以下已知限制：

* 应用于 `SurfaceView` 的动画可能无法正确显示。`SurfaceView` 从非 UI 线程中更新 UI，因此更新可能与其他视图的动画不同步
* 当应用于 `TextureView` 时，某些特定的 Transition 可能不会产生所需的动画效果
* 继承于 `AdapterView` 的类（比如 `ListView` ）以同框架不兼容的方式管理子视图。如果您尝试设置动画 ，设备显示可能会出错。
* 如果您尝试使用动画调整 `TextView` 的大小，则在对象完全调整好大小之前，文本内容可能已经改变到新位置了。为了避免这个问题，请不要将动画应用在改变包含文本内容的视图大小上。