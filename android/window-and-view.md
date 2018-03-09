本文的意义是为了能理清 Android 中 Window 和 View 的关系。



### 定义

`android.view.Window` 的定义是顶级窗口外观和行为策略的抽象基类。类的实例应该被作为顶级视图添加到 window manager。它提供了标准的 UI 策略，比如背景，标题区域，默认按键处理，等等。

上面这段介绍是 API 文档中的介绍，简单来说，Window 是顶级视图的抽象表示。

`android.view.View` 的定义是表示用户界面组件的基础构建块。简单来说，View 是 Android 绘制用户界面的最小元素。

从上面两个介绍，可以简单认为 Window 是用户界面的抽象表示，其中包含标准的用户界面外观定义和与用户进行交互的行为策略等等，但是具体的实现应该是交与 View。View 负责在 canvas 上绘制用户真正看见的像素。



### 源码

Activity 是 Android 用户界面的载体，每个 Activity 实例都可以通过 `getWindow` 获取 Window 实例，那么可以通过 Activity 的启动流程，来了解 Activity 中 Window 的实例的初始化等等。

我们知道启动一个 Activity 的流程非常复杂，从 Context 会调用到 ActivityManagerService，然后调回到最重要的 ActivityThead，题外话，虽然这个类名字中有 Thread，但它并不是一个 Thread 的子类，而它执行 `main` 方法的线程，就是我们常说的 UI 线程。现在我们只需要知道的是，最终会调用到 ActivityThread 的 `handleLaunchActivity` 方法，在这个方法中去创建 Activity 的实例，并且执行它的生命周期。

``` java
private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent, String reason) {      
    // 省略无关主题的代码
    // 这里会去创建 Activity 的实例，并执行一部分生命周期
    Activity a = performLaunchActivity(r, customIntent);
    if (a != null) {
        // 调用 Activity.onResume
         handleResumeActivity(r.token, false, r.isForward,                                 
         !r.activity.mFinished && !r.startsNotResumed, r.lastProcessedSeq, reason);
    }
}                                                                                                    
```

而 `performLaunchActivity` 方法中调用 Activity 的 `attach` 方法，在这个方法中去完成 Window 的初始化。

``` java
final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback) {
	
    // 可以看到 Window 的实现类是 PhoneWindow
    mWindow = new PhoneWindow(this, window, activityConfigCallback);
    
}
```

经过上面的流程，我们可以知道 Activity 中的 Window 实例化是在 `Activity.attach` 方法中。

接下来，我们可以先看下 `handleLaunchActivity` 方法中调用的 `handleResumeActivity` 对 Window 执行了一些什么操作。

``` java
final void handleResumeActivity(IBinder token,
            boolean clearHide, boolean isForward, boolean reallyResume, int seq, String reason) {
    
    View decor = r.window.getDecorView();
    ViewManager wm = a.getWindowManager();
    WindowManager.LayoutParams l = r.window.getAttributes();
    // 记住这个方法调用
    wm.addView(decor, l);
}
```

现在我们只需要先知道在调用 Activity 的 `onResume` 方法时会调用 WindowManager 的 `addView` 方法。



