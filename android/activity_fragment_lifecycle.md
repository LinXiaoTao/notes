### Activity LifeCycle

![activity_lifecycle](https://developer.android.com/images/activity_lifecycle.png?hl=zh-cn)

* onCreate(Bundle)

  首次创建 Activity 时调用。系统向此方法传递一个 Bundle 对象，其中包含 Activity 的上一状态，不过前提是捕获了该状态

* onRestart()

  在 Activity 已停止并即将再次启动前调用。

* onStart()

  在 Activity 即将对用户可见之前调用。如果 Activity 转入前台，则后接 `onResume()`，如果 Activity 转入隐藏状态，则后接 `onStop()`。

* onResume()

  在 Activity 即将开始与用户进行交互之前调用。 此时，Activity 处于 Activity 堆栈的顶层，并具有用户输入焦点。始终后接 `onPause()`。

* onPause()

  当系统即将开始继续另一个 Activity 时调用。如果 Activity 返回前台，则后接 `onResume()`，如果 Activity 转入对用户不可见状态，则后接 `onStop()`。

* onStop()

  在 Activity 对用户不再可见时调用。如果 Activity 恢复与用户的交互，则后接 `onRestart()`，如果 Activity 被销毁，则后接`onDestroy()`。

* onDestory()

  在 Activity 被销毁前调用。这是 Activity 将收到的最后调用。

> `onCreate(Bundle)` `onStart()` `onResume()` `onRestart()` 在正常情况下，系统不能在不执行另一行 Activity 代码的情况下，在*方法返回后*随时终止承载 Activity 的进程
>
> 而 `onPause()` `onStop()` `onDestory()` 则反之。



### 保存 Activity 状态

![activity_state](https://developer.android.com/images/fundamentals/restore_instance.png?hl=zh-cn)

系统会先调用 `onSaveInstanceState()`，然后再使 Activity 变得易于销毁。系统会向该方法传递一个 `Bundle`，您可以在其中使用 `putString()` 和 `putInt()` 等方法以名称-值对形式保存有关 Activity 状态的信息。然后，如果系统终止您的应用进程，并且用户返回您的 Activity，则系统会重建该 Activity，并将 `Bundle` 同时传递给 `onCreate()` 和 `onRestoreInstanceState()`。

> **注**：无法保证系统会在销毁您的 Activity 前调用 `onSaveInstanceState()`，因为存在不需要保存状态的情况（例如用户使用“返回”按钮离开您的 Activity 时，因为用户的行为是在显式关闭 Activity）。 如果系统调用 `onSaveInstanceState()`，它会在调用 `onStop()` 之前，并且可能会在调用 `onPause()` 之前进行调用。

不过，即使您什么都不做，也不实现 `onSaveInstanceState()`，`Activity` 类的 `onSaveInstanceState()` 默认实现也会恢复部分 Activity 状态。具体地讲，默认实现会为布局中的每个 `View` 调用相应的 `onSaveInstanceState()` 方法，让每个视图都能提供有关自身的应保存信息。只需为想要保存其状态的每个小部件提供一个唯一的 ID（通过 [`android:id`](https://developer.android.com/guide/topics/resources/layout-resource.html?hl=zh-cn#idvalue) 属性）。如果小部件没有 ID，则系统无法保存其状态。

由于 `onSaveInstanceState()` 的默认实现有助于保存 UI 的状态，因此如果您为了保存更多状态信息而替换该方法，应始终先调用 `onSaveInstanceState()` 的超类实现，然后再执行任何操作。 同样，如果您替换 `onRestoreInstanceState()` 方法，也应调用它的超类实现，以便默认实现能够恢复视图状态。



### Fragment LifeCycle

![fragment_lifecycle](https://developer.android.com/images/fragment_lifecycle.png?hl=zh-cn)

* onCreate(Bundle)

  系统会在创建片段时调用此方法。您应该在实现内初始化您想在片段暂停或停止后恢复时保留的必需片段组件。

* onCreateView()

  系统会在片段首次绘制其用户界面时调用此方法。 要想为您的片段绘制 UI，您从此方法中返回的 `View` 必须是片段布局的根视图。如果片段未提供 UI，您可以返回 null。

* onPause()

  系统将此方法作为用户离开片段的第一个信号（但并不总是意味着此片段会被销毁）进行调用。



### 处理 Fragment 生命周期

![Activity 生命周期对片段生命周期的影响](https://developer.android.com/images/activity_fragment_lifecycle.png?hl=zh-cn)

* onAttach()

  在片段已与 Activity 关联时调用（`Activity` 传递到此方法内）。

  > 在这个方法中，Activity 可能还没创建完成。Activity 正确创建完成时间为：`onActivityCreated()`

* onCreateView()

  调用它可创建与片段关联的视图层次结构。

* onActivityCreated()

  在 Activity 的 `onCreate()` 方法已返回时调用。

* onDestroyView()

  在移除与片段关联的视图层次结构时调用。

* onDetach()

  在取消片段与 Activity 的关联时调用。

> 通常情况下，在 A Activity 开始 B Activity，回调方法顺序大致如下：
>
> ``` java
> A.onPause();
> B.onCreate();
> B.onStart();
> B.onResume();
> A.onSaveInstanceState();
> A.onStop();
> ```