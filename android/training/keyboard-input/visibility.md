[参考](https://developer.android.com/training/keyboard-input/visibility.html?hl=zh-cn)

# 处理输入法的可见性

当输入焦点从可编辑文本字段移入或移出时，Android 会适当地显示或隐藏输入法。系统还会决定您的 UI 和文本字段在输入法上方的显示方式。例如，当屏幕上的垂直空间受到限制时，文本字段可能会填充输入法上方的所有空间。对于大多数应用，这些默认行为是需要的。



## 在 Activity 启动时显示输入法

``` xml
<application ... >
    <activity
        android:windowSoftInputMode="stateVisible" ... >
        ...
    </activity>
    ...
</application>
```



## 在需要的时候显示输入法

``` java
public void showSoftKeyboard(View view) {
    if (view.requestFocus()) {
        InputMethodManager imm = (InputMethodManager)
                getSystemService(Context.INPUT_METHOD_SERVICE);
        imm.showSoftInput(view, InputMethodManager.SHOW_IMPLICIT);
    }
}
```



## 指定界面如何响应输入法

当屏幕出现输入法时，会减少UI界面的可用空间。

使用 `adjustResize`，系统将布局调整到可用空间，确保所有布局内容都可见

``` xml
<application ... >
    <activity
        android:windowSoftInputMode="adjustResize" ... >
        ...
    </activity>
    ...
</application>
```

可以和输入法可见性结合使用

``` xml
<activity
        android:windowSoftInputMode="stateVisible|adjustResize" ... >
        ...
 </activity>
```