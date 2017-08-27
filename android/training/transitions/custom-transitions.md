[参考](https://developer.android.com/training/transitions/custom-transitions.html)

# 创建自定义 Transitions

自定义 transition 将动画应用于开始和结束 scene 的子视图。但是，于内置 transition 类型不同的是，您必须提供捕获属性值并生成动画的代码。您可能还需要为动画定义目标视图的一个子集。



## 扩展 Transition 类

``` java
public class CustomTransition extends Transition {

    @Override
    public void captureStartValues(TransitionValues values) {}

    @Override
    public void captureEndValues(TransitionValues values) {}

    @Override
    public Animator createAnimator(ViewGroup sceneRoot,
                                   TransitionValues startValues,
                                   TransitionValues endValues) {}
}
```



## 捕获视图属性值

transiton 动画使用属性动画系统。框架提供回调方法，允许 transition 仅捕获所需的属性值，并将其存储在框架中。



### 捕获起始属性值

要将起始视图属性值传递给框架，请执行 `captureStartValues(transitionValues)` 方法。`transitionValues` 是一个 `TransitionValues` 对象，它包含对视图的引用和一个 `Map` 实例，您可以在其中存储所需的属性值。在您的实现中，检索这些属性值，并通过将它们存储在 `Map` 中将其传递给框架。

要确保属性值的键不与其他 `TransitionValues` 的键冲突，请使用以下命名方案：

`package_name:transition_name:property_name`

``` java
public class CustomTransition extends Transition {

    // Define a key for storing a property value in
    // TransitionValues.values with the syntax
    // package_name:transition_class:property_name to avoid collisions
    private static final String PROPNAME_BACKGROUND =
            "com.example.android.customtransition:CustomTransition:background";

    @Override
    public void captureStartValues(TransitionValues transitionValues) {
        // Call the convenience method captureValues
        captureValues(transitionValues);
    }


    // For the view in transitionValues.view, get the values you
    // want and put them in transitionValues.values
    private void captureValues(TransitionValues transitionValues) {
        // Get a reference to the view
        View view = transitionValues.view;
        // Store its background property in the values map
        transitionValues.values.put(PROPNAME_BACKGROUND, view.getBackground());
    }
    ...
}
```



### 捕获结束属性值

``` java
@Override
public void captureEndValues(TransitionValues transitionValues) {
    captureValues(transitionValues);
}
```

`captureEndValues()` 检索的视图属性值是相同的，但在起始和结束 scene 中具有不同的值。框架为视图的开始和结束状态维护单独的映射。



## 创建自定义动画

可以通过覆盖 `createAnimator()` 方法来提供动画。当框架调用这个方法时，它会传递 scene root 视图和 `TransitionValues` 对象，这个对象保存您捕获的起始属性值和结束属性值。

框架调用 `createAnimator()` 方法的次数取决于开始和结束 scene 之间发生的更改。例如，考虑定义淡入淡出的 transiton，如果起始 scene 由五个目标，其中两个目标将从结束 scene 中删除，并且结束 scene 在起始 scene 剩下的三个目标上再加上一个新的目标，则框架会调用 `createAnimator()` 六次：三次是两个 scene 同时存在的三个目标的淡入淡出，另外两个是从结束 scene 中删除的两个目标的淡出，剩下的一个是结束 scene 中新目标的淡入。

对于起始和结束 scene 都存在的目标视图，框架在 `startValues` 和 `endValues` 参数都提供了一个 `TransitionValues` 对象。对于仅存在一个 scene 的目标视图，则只有相应的 `TransitionValues` 对象。



## 应用自定义 Transition

自定义 transition 与内置的 transition 工作相同，可以通过使用 transition manager 应用自定义 transition。