典型的 `finishAfterTransition` 是这样的：

1. `finishAfterTransition()` 创建一个 `ExitTransitionCoordinator`，并且调用 `startExit()`。
   * window 开始过渡到透明，并且有个新的 `ActivityOptions`。
   * 如果没有背景，那么就会使用一个黑色背景代替。
   * 共享元素进行匹配。
   * 将视图设置为 `INVISIBLE` 开始退出转场。
2. 返回的 Activity 接收到 `ActivityOptions`，同时 `EnterTransitionCoordinator` 被创建。
   * 所有的转场视图设置为 `VISIBLE`，这和当 `onActivityStopped()` 被调用时所做的事情相反。
3. window 为透明，并且回调被接收。
   * 背景透明度动画过渡至 0。
4. 背景透明度动画完成。
5. 共享元素转场完成。
   * 在步骤 4 和步骤 5 完成后，`MSG_TAKE_SHARED_ELEMENTS` 被发送至 `EnterTransitionCoordinator`。
6. `EnterTransitionCoordinator` 接收到 `MSG_TAKE_SHARED_ELEMENTS` 通知。
   * 共享元素设置为 `VISIBLE`。
   * 共享元素的位置和尺寸和调用 Activity 的结束状态相匹配
   * 共享元素转场开始。
   * 如果 window 允许转场覆盖，那么 content 转场也会通过设置进入视图为 `VISIBLE` 开始。
   * `MSG_HIDE_SHARED_ELEMENTS` 被发送至 `ExitTransitionCoordinator`。
7. `ExitTransitionCoordinator` 接收到 `MSG_HIDE_SHARED_ELEMENTS` 通知。
   * 将共享元素设置为 `INVISIBLE`。
8. 即将结束的 Activity 退出转场完成。
   * 向 `EnterTransitionCoordinator` 发送 `MSG_EXIT_TRANSITION_COMPLETE` 通知。
   * 退出 Activity 调用 `finish()`。
9. `EnterTransitionCoordinator` 接收到 `MSG_EXIT_TRANSITION_COMPLETE` 通知。
   * 如果 window 不允许进入转场覆盖，那么进入转场通过设置视图为 `VISIBLE` 开始。



> 总结：
>
> 1. 当调用 `finishAfterTransition()` 时，如果没有需要退出转场，则立刻调用 `finish()`，否则，则调用 `startExitBackTransition()`，方法首先取消进入转场，同时创建一个 `ExitTransitionCoordinator`，如果已有进入转场正在进行，则等待至 `onPreDraw`，否则直接调用 `ExitTransitionCoordinator.startExit(int,Intent)`。
> 2. `ExitTransitionCoordinator.startExit(int,Intent)` 会阻塞输入，抑制 layout，延迟发送取消事件，将共享元素移动至 `ViewOverlay`，如果 decorView 不存在，或者没有背景，则 windown 背景设置为透明，同时调用 `ActivityOptions.makeSceneTransitionAnimation() `生成一个新的 `ActivityOptions`，这个方法会将通过 `setResult()` 的参数包装到 `ActivityOptions` 中。将 Activity 转换为透明，并在转换完成后，调用 `ExitTransitionCoordinator.fadeOutBackgroud()`，接下来，调用 `starttransition()` 执行 `startExitTransition()` 开始进行退出转场。
> 3. `ExitTransitionCoordinator.fadeOutBackground()` 中会执行一个动画，将 window 的背景透明度设置为 0，当动画结束后，则调用 `ExitTransitionCoordinator.notifyComplete()`，等待 `EnterTransitionCoordinator` 的通知。
> 4. `ExitTransitionCoordinator.startExitTransition()` 开始执行退出转场动画，通过将转场视图从 `VISIBLE` 过渡为 `INVISIBLE`，并在转场结束后调用 `viewsTransitionComplete()`。当转场完成后，则会调用至 `ExitTransitionCoordinator.notifyComplete()`，等待 `EnterTransitionCoordinator` 的通知。
> 5. 当返回的 Activity Restart 时，会调用 `ActivityTransitionState.enterReady()` 创建 `EnterTransitionCoordinator`，构造方法中如果不是跨 Task 的 Activity 则去除默认动画，设置 Activity 为透明，背景为透明，发送 `MSG_SET_REMOTE_RECEIVER` 给 `ExitTransitionCoordinator`。如果没有延迟进入转场，则调用 `startEnter()`。
> 6. `ActivityTransitionState.startEnter()` 会调用至 `EnterTransitionCoordinator.sendSharedElementDestination()`，这里会在 onPreDraw 中去捕获共享元素的属性，并且将共享元素移动至 Viewoverlay，然后将属性通过 `MSG_SHARED_ELEMENT_DESTINATION` 发送给 `ExitTransitionCoordinator`，如果允许转场之间覆盖，则调用 `EnterTransitionCoordinator.startEnterTransitionOnly()` 执行 content 转场。
> 7. `EnterTransitionCoordinator.startEnterTransitionOnly()` 通过调用 `EnterTransitionCoordinator.beginTransition()` 开始执行 content 转场，并在转场结束后调用 `viewTransitionComplete()`。同时调用 `EnterTransitionCoordinator.startEnterTransition()` 将 window 背景透明度动画过渡为 255，设置 Activity 不透明。
> 8. `ExitTransitionCoordinator` 接收到 `MSG_SHARED_ELEMENT_DESTINATION` 通知后，会调用至 `ExitTransitionCoordinator.startSharedElementExit()` 开始共享元素退出转场，通过应用从 `EnterTransitionCoordinator` 发送的共享元素状态来进行动画。
> 9. 当 `ExitTransitionCoordinator` 完成 content 转场和共享元素转场后，会发送 `MSG_TAKE_SHARED_ELEMENTS` 通知给 `EnterTransitionCoordintor` 进行共享元素转场，接着发送 `MSG_EXIT_TRANSITION_COMPLETE` 通知 `EnterTransitionCoordintor` 退出转场结束。
> 10. 当返回的 Activity Resume 时，不管转场是否完成，强制所有视图都出现。同时也将退出的 Activity 的视图设置为 `VISIBLE`。
> 11. 当进入转场完成时（`EnterTransitionCoordinator.onTransitionComplete()`），将共享元素从 ViewOverlay 移动回来。

> `ExitTransitionCoordinator` 先进行 content 转场，等待 `EnterTransitionCoordinator` 发送 `MSG_SHARED_ELEMENT_DESTINATION` 通知，进行共享元素转场。
>
> `EnterTransitionCoordinator` 在 Activity ReStart 时，发送 `MSG_SHARED_ELEMENT_DESTINATION` 通知，同时如果允许转场覆盖，则开始 content 转场。
>
> `ExitTransitionCoordinator` 接收 `MSG_SHARED_ELEMENT_DESTINATION` 通知，进行共享元素转场（主要），发送 `MSG_TAKE_SHARED_ELEMENTS` 通知，如果 content 转场和共享元素转场都完成，发送 `MSG_EXIT_TRANSITION_COMPLETE` 通知。
>
> `EnterTransitionCoordinator` 接收 `MSG_TAKE_SHARED_ELEMENTS` 通知进行共享元素转场。接收到 `MSG_EXIT_TRANSITION_COMPLETE` 通知，如果需要，开始 content 转场。