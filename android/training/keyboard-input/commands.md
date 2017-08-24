[参考](https://developer.android.com/training/keyboard-input/commands.html?hl=zh-cn)

# 处理键盘操作

当用户将焦点放在可编辑的文本视图（例如 EditText 元素上），并且用户附加了硬件键盘，所有的输入都由系统处理。然而，如果您想去自己去拦截或者直接处理键盘输入，您可以实现 [KeyEvent.Callback](https://developer.android.com/reference/android/view/KeyEvent.Callback.html?hl=zh-cn) 接口的回调方法，比如 `onKeyDown()` 和 `onKeyMultiple()`。

Activity 和 View 都实现了 `KeyEvent.Callback` 接口，所以您应该覆盖这些类的回调方法。

> **注意：**当您使用 `KeyEvent` 类和有关 API 来处理键盘事件时，您应该知道所有事件至来源硬件键盘。



## 处理单个按键

通常，如果要确保您只收到一个事件，您应该使用 `onKeyUp()`。如果用户按住按钮，那么 `onKeyUp()` 被多次调用。

``` java
@Override
public boolean onKeyUp(int keyCode, KeyEvent event) {
    switch (keyCode) {
        case KeyEvent.KEYCODE_D:
            moveShip(MOVE_LEFT);
            return true;
        case KeyEvent.KEYCODE_F:
            moveShip(MOVE_RIGHT);
            return true;
        case KeyEvent.KEYCODE_J:
            fireMachineGun();
            return true;
        case KeyEvent.KEYCODE_K:
            fireMissile();
            return true;
        default:
            return super.onKeyUp(keyCode, event);
    }
}
```



## 处理多个按键

几种方法提供有关修饰键的信息，如 `getModifiers()` 和 `getMetaState()`。然而，最简单的解决方法是检查您关心的确切修饰符键是否被诸如 `isShiftPressed()` 和 `isCtrlPressed()` 等等。

``` java
@Override
public boolean onKeyUp(int keyCode, KeyEvent event) {
    switch (keyCode) {
        ...
        case KeyEvent.KEYCODE_J:
            if (event.isShiftPressed()) {
                fireLaser();
            } else {
                fireMachineGun();
            }
            return true;
        case KeyEvent.KEYCODE_K:
            if (event.isShiftPressed()) {
                fireSeekingMissle();
            } else {
                fireMissile();
            }
            return true;
        default:
            return super.onKeyUp(keyCode, event);
    }
}
```

