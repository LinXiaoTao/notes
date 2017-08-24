[参考](https://developer.android.com/training/keyboard-input/navigation.html?hl=zh-cn)

# 支持键盘导航

除了软键盘（如屏幕键盘）以外，Android 还支持连接到设备的物理键盘。键盘不仅提供了方便的文本输入，还为用户提供导航来与您的应用进行互动。虽然大多数手持设备如手机，使用触摸作为互动的主要模式，但平板电脑和类似设备越来越受欢迎，许多用户喜好附加键盘配件。

> **注意：**在应用中支持定向导航，可以确保不使用视觉导航的用户也能 [访问](https://developer.android.com/guide/topics/ui/accessibility/apps.html?hl=zh-cn)，还可以帮助您通过 [uiautomator](https://developer.android.com/tools/help/uiautomator/index.html?hl=zh-cn) 等工具进行 [UI 测试](https://developer.android.com/tools/testing/testing_ui.html?hl=zh-cn)。



## 测试您的应用



## 处理 TAB 导航

当用户使用键盘Tab键导航您的应用程序时，系统会根据它们在布局中显示的顺序，在元素之间传递输入焦点。

可以使用 `android:nextFocusForward` 来指定确认获取焦点的顺序。



## 处理定向导航

用户还可以使用键盘上的箭头键导航您的应用（行为与使用 D-pad 或者轨迹球类似）。系统提供了一个最好的猜测，即根据屏幕上的视图布局，然而系统可能会猜测错误。

可以使用以下属性定义定向导航焦点顺序

* [android:nextFocusUp](https://developer.android.com/reference/android/view/View.html?hl=zh-cn#attr_android:nextFocusUp)
* [android:nextFocusDown](https://developer.android.com/reference/android/view/View.html?hl=zh-cn#attr_android:nextFocusDown)
* [android:nextFocusLeft](https://developer.android.com/reference/android/view/View.html?hl=zh-cn#attr_android:nextFocusLeft)
* [android:nextFocusRight](https://developer.android.com/reference/android/view/View.html?hl=zh-cn#attr_android:nextFocusRight)

