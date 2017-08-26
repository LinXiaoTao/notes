[参考](https://developer.android.com/training/animation/layout.html)

# 布局改变动画

布局动画是每次更改布局配置时，系统运行的预加载动画。

> **提示：**如果要提供自定义布局动画，请创建一个 [LayoutTransition](https://developer.android.com/reference/android/animation/LayoutTransition.html) 对象，并通过 `setLayouttransition()` 应用在布局中。



## 创建布局

``` xml
<LinearLayout android:id="@+id/container"
    android:animateLayoutChanges="true"
    ...
/>
```



## 从布局中添加、更新或者删除子项

``` java
private ViewGroup mContainerView;
...
private void addItem() {
    View newView;
    ...
    mContainerView.addView(newView, 0);
}
```

