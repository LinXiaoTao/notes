### SYSTEM_UI_FLAG

* SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN

  设置这个标记后，即使当前没有设置 **SYSTEM_UI_FLAG_FULLSCREEN**，视图也会如同被设置了 FULLSCREEN 标记一样去布局。它可以避免在切换和退出该模式时的伪影，不过其代价是一些用户界面可能被屏幕装饰（ActioBar 或者 状态栏等等）覆盖。可以通过 `fitSystemWindows(Rect)` 方法执行内部 UI 元素布局来解决。

* SYSTEM_UI_FLAG_LAYOUT_STABLE

  当使用其他布局标记时，我们想要一个稳定的视图来 `fitSystemWindows(Rect)` 。这意味着