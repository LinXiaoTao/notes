[参考](https://developer.android.com/training/transitions/scenes.html)

# 创建 Scene

Scene 保存一个视图层次结构的状态，包含这个视图层次结构的所有的视图和它们的属性值。

起始 Scene 总是自动从用户界面上获取当前的视图层次结构状态。

> **注意：**框架可以在不使用 scenes 的情况下，将动画效果作用在单个视图层次结构的改变中。



## 从布局资源文件中创建 Scene

当文件中的视图结构层次大部分时静态时，使用从布局资源文件中创建 Scene。生成的这个 Scene 表示当您生成 Scene 实例时视图层次结构的状态。如果更改视图层次结构，则必须重新创建 Scene。框架从文件中的整个视图层次结构创建 Scene；您无法使用布局文件的一部分来创建 Scene。

从布局资源文件中创建 Scene，然后从您的布局中检索一个 `ViewGroup` 实例作为 scene root，同时将 scene root 和布局文件的 resource ID 作为参数，来调用 `Scene.getSceneForLayout()` 方法，从而返回包含视图层次结构的 scene。



## 定义 Scenes 布局

下面这些代码片段介绍如何使用相同的 scene root 创建两个不同的 scene，并且可以加载多个不相关的 scene 对象，而不意味着它们彼此相关。



`activity_main.xml`

``` xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/master_layout">
    <TextView
        android:id="@+id/title"
        ...
        android:text="Title"/>
    <FrameLayout
        android:id="@+id/scene_root">
        <include layout="@layout/a_scene" />
    </FrameLayout>
</LinearLayout>
```



`a_scene.xml`

``` xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scene_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/text_view1
        android:text="Text Line 1" />
    <TextView
        android:id="@+id/text_view2
        android:text="Text Line 2" />
</RelativeLayout>
```



第二个 scene 包含两个相同的文本（相同的 ID），但放置顺序不同。

`another_scene.xml`

``` xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scene_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/text_view2
        android:text="Text Line 2" />
    <TextView
        android:id="@+id/text_view1
        android:text="Text Line 1" />
</RelativeLayout>
```



### 从布局中生成 Scenes

下面的代码片段展示如何获取对 scene root 的引用，并从布局文件中创建两个 `Scene` 对象。

``` xml
Scene mAScene;
Scene mAnotherScene;

// Create the scene root for the scenes in this app
mSceneRoot = (ViewGroup) findViewById(R.id.scene_root);

// Create the scenes
mAScene = Scene.getSceneForLayout(mSceneRoot, R.layout.a_scene, this);
mAnotherScene =
    Scene.getSceneForLayout(mSceneRoot, R.layout.another_scene, this);
```



## 在您的代码中创建 Scene

您可以在您的代码中，从一个 `ViewGroup` 对象中创建 `Scene` 实例。在代码中直接修改视图层次结构或动态生成视图层次结构时，请使用这种方法。

使用 `Scene(sceneRoot,viewHierarchy)` 构造方法，在您的代码中从视图层次结构中创建一个 scene，这相对于您已经 inflated 布局文件后，调用 `Scene.getSceneForLayout()`。

``` java 
Scene mScene;

// Obtain the scene root element
mSceneRoot = (ViewGroup) mSomeLayoutElement;

// Obtain the view hierarchy to add as a child of
// the scene root when this scene is entered
mViewHierarchy = (ViewGroup) someOtherLayoutElement;

// Create a scene
mScene = new Scene(mSceneRoot, mViewHierarchy);
```



## 创建 Scene Action

框架使您能够定义系统在进入 scene 或者退出 scene 时运行的自定义 scene action。在大部分情况下，框架自动给 scenes 变化添加动画效果。

Scene Action 对于处理这些情况很有用：

* 动画作用在不属于同一层次结构中的视图。您可以使用退出和进入 Scene Action 为起始 scenes 和结束 scenes 添加动画效果。
* 框架不能自动添加动画效果的视图，比如 `ListView` 对象等等。

要提供自定义的 Scene Action，请将您的 action 定义为 `Runnable` 对象，并将它传递给 `Scene.setExitAction()` 或者 `Scene.setEnterAction()` 方法。框架在运行 transition 动画之后，在结束 scene 中运行 transition 动画和 `setEnterAction` 之前，先在起始 scene 中调用 `setExitAction()` 方法。

> **注意：**不要在 scene action 在 开始和结束 scene 中的视图之间传递数据。

