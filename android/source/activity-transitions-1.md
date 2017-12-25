典型的 `startActivity` 类似：

1. 通过 `ActivityOptions.makeSceneTransitionAnimation()` 创建 `ExitTransitionCoordinator`

2. `Activity.startActivity` 被调用，同时通过 ~~`ActivityOptions.dispacthStartExit`~~ `ActivityTransitionState.startExitOutTransition` 调用 `startExit()`

   > Exit transition 首先将转场的视图设置为 `INVISIBLE`

3. 启动 Activity，创建一个 `EnterTransitionCoordinator`。

   * Window 设置为透明
   * Window 背景透明度设置为 0
   * 转场视图设置为 INVISIBLE
   * 发送 ~~`MSG_SET_LISTENER`~~`MSG_SET_REMOTE_RECEIVER` 给 `ExitTransitionCoordinator`。

4. 共享元素转场完成。

   * 向 `EnterTransitionCoordinator` 发送 `MSG_TAKE_SHARED_ELEMENTS`

5. 当 `EnterTransitionCoordinator` 接收到 `MSG_TAKE_SHARED_ELEMENTS`。

   * 设置共享元素为 `VISIBLE`。
   * 将共享元素的位置的位置和尺寸设置和调用 Activity 匹配。
   * 开始共享元素转场。
   * 如果 window 允许转场互相覆盖 `android:windowAllowEnterTransitionOverlap` 或 `android:windowAllowReturnTransitionOverlap`，那么视图转场开始于将进入视图设置为 `VISIBLE` 和背景透明度动画变化至不透明。



6. 当 `ExitTransitionCoordinator` 收到 `MSG_HIDE_SHARED_ELEMENTS`

   * 将共享元素视图设置为 `INVISIBLE`

7. 调用 Activity 的退出转场完成。

   * 向 `EnterTransitionCoordinator` 发送 `MSG_EXIT_TRANSITION_COMPLETE`
8. `EnterTransitionCoordinator` 接收到 `MSG_EXIT_TRANSITION_COMPLETE`

   * 如果 window 不允许和进入转场重叠，那么进入转场通过设置进入视图为 `VISIBLE`，同时背景动画变化至不透明来开始。
9. 背景不透明动画完成的

   * window 设置为不透明
10. 调用 Activity 获取一个 `onStop` 的调用，然而调用 ~~`onActivityStopped()`~~`ActivityTransitionState.onStop`，同时将所有退出的视图设置为 `VISIBLE`。 



> 总结：
>
> 1. 每个 Activity 都持有一个 `ActivityTransitionState`，通过它来进行 Transition 活动。
> 2. `ActivityOptions.makeSceneTransitionAnimation()` 会生成 `ExitTransitionCoordinator`，`ExitTransitionCoordinator`  构造方法中会调用 `ActivityTransitionCoordinator.viewsReady()`  回调 `SharedElementCallback.onMapSharedElements()`，同时配置共享元素视图，设置第一个共享视图为转场的中心。接着调用 `ActivityTransitionCoordinator.stripOffscreenViews()` 移除不在屏幕范围之内的转场视图。`ActivityOptions.makeSceneTransitionAnimation()` 方法在实例化 `ExitTransitionCoordinator` 后会调用 `ActivityTransitionState.makeSceneTransitionAnimation()`，将 `ExitTransitionCoordinator` 添加到 `ActivityTransitionState` 的软引用中。
> 3. 当调用 `Activity.startActivity()` 时，会调用至 `ActivityTransitionState.startExitOutTransition()`，先判断是否开启 Activity 转场，从软引用中获取 `ExitTransitionCoordinator` 的引用，从而调用 `ExitTransitionCoordinator.startExit()`。
> 4. `ExitTransitionCoordinator.startExit()` 设置 `mBackgroundAnimatorComplete = true`，阻塞输入，抑制 layout，将共享元素移动至 ViewOverlay，之后调用 `ExitTransitionCoordinator.beginTransitions()`，这里会过滤到一些无效视图，共享元素转场和内容转场之间重叠的视图等等，然后开始进行转场动画，将视图从可见到不可见。当共享元素转场结束后，捕获当前共享元素的属性，如果内容转场也结束了，则调用至 `ExitTransitionCoordinator.notifyComplete()`，等待 `EnterTransitionCoordinator` 的通知。
> 5. 当目标 Activity 创建时，会调用至 `ActivityTransitionState.setEnterActivityOptions()`，然后调用 `ActivityTransitionState.enterReady()` 创建 `EnterTransitionCoordinator`，构造方法中如果不是跨 Task 的 Activity 则去除默认动画，设置 Activity 为透明，背景为透明，发送 `MSG_SET_REMOTE_RECEIVER` 给 `ExitTransitionCoordinator`。
> 6. 当目标 Activity resume 时，会调用 `ActivityTransitionState.onResume()` ，`restoreExitedViews()` 和 `restoreReenteringViews()`。
> 7. `ActivityTransitionStae.startEnter()` 开始执行进入转场，调用至 `EnterTransitionCoordinator.viewsReady()`，这个方法会捕获所需转场视图属性，过滤掉和共享元素视图重叠的，在屏幕之外的，已排除的视图，然后调用 `hideViews()` 将转场视图设置为透明，将共享元素移动到 ViewOverlay。
> 8. `ExitTransitionCoordinator` 接收到 `MSG_SET_REMOTE_RECEIVER` 通知，将共享元素视图属性 `MSG_TAKE_SHARED_ELEMENTS` 发送给 `EnterTransitionCoordinator`，并且如果退出转场已结束，则发送 `MSG_EXIT_TRANSITION_COMPLETE` 通知，如果有需要则关闭当前 Activity。
> 9. `EnterTransitionCoordinator` 接收到 `MSG_TAKE_SHARED_ELEMENTS` 通知，调用至 `startSharedElementTransition()` 开始共享元素转场，先删除拒绝共享元素，将这些元素透明度设置为 0，然后调用 `showViews()` 复原共享元素视图的透明度，调用 `setSharedElementState()` 保存目标 Activity 的共享元素视图状态，应用调用 Activity 的共享元素视图状态，然后 `requestLayoutForSharedElements()` 遍历调用共享元素视图的 `requestLayout()`，调用 `beginTransition()` 应用转场动画将转场视图从 `INVISIBLE` 设置为 `VISIBLE`。
> 10. 当共享元素转场和内容转场都完成了，调用到 `onTransitionsComplete()` 将共享元素从 ViewOverlay 移动回来，同时 `ExitTransitionCoordinator` 会接受到 `MSG_HIDE_SHARED_ELEMENTS`，隐藏共享元素视图，在需要情况下，关闭当前 Activity。

>`startActivity()` 时调用 `ExitTransitionCoordinator.beginTransitions()` 开始进行 content 转场和共享元素转场。完成后，发送 `MSG_TAKE_SHARED_ELEMENTS` 通知，并且携带共享元素状态。接着发送 `MSG_EXIT_TRANSITION_COMPLETE` 通知。
>
>`EnterTransitionCoordinator` 接收到 `MSG_TAKE_SHARED_ELEMENTS` 通知后，调用 `EnterTransitionCoordinator.beginTransition()` 开始共享元素转场，并且发送 `MSG_HIDE_SHARED_ELEMENTS` 通知。如果允许转场覆盖，则同时开始进入转场。
>
>`EnterTransitionCoordinator` 开始共享元素转场，如果 `ExitTransitionCoordinator` 已经结束转场，则向自己发送 `MSG_EXIT_TRANSITION_COMPLETE`，是否需要启动进入转场。
>
>`ExitTransitionCoordinator` 接收到 `MSG_HIDE_SHARED_ELEMENTS` 通知后，隐藏共享元素。