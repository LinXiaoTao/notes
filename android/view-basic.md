### Themes

默认情况下，View 使用构造方法中的 Context 中的 Theme。然而，可以通过在 XML 中使用 `android:theme` 属性或者在构造方法中使用 [ContextThemeWrapper](https://developer.android.com/reference/android/view/ContextThemeWrapper.html) 来使用不同的 Theme。

当在 XML 中 `android:theme` 属性被使用，指定的 Theme 将被应用于 Context 的主题之上（[LayoutInflater](https://developer.android.com/reference/android/view/LayoutInflater.html)），并且被当前 View 及其子元素使用。

``` xml
     <LinearLayout
             ...
             android:theme="@android:theme/ThemeOverlay.Material.Dark">
         <View ...>
     </LinearLayout>
```

> Android 5.0，Theme 和 ThemeOverlay 的区别：
>
> ThemeOverlay 是重叠普通 Theme 的特殊 Theme，覆盖相关部分属性。

