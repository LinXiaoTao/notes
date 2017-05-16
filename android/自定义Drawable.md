### 自定义 Drawable

Drawable 是 **可绘制对象** 的抽象，与 `android.view.View` 不同，Drawable 没有任何功能接收事件或其他方式与用户交互。

Android 的所有版本都允许在运行时扩展和使用 `Drawable` 类代替框架提供的 `drawable` 类。在 API 24 开始，可以在 XML 中使用自定义 Drawable 。

注意：自定义 Drawable 只能在应用程序中访问，其他应用程序将无法加载它们。

自定义 Drawable 至少必须在 Drawable 上实现抽象方法，并且应该覆盖 `draw(Canvas)` 方法来绘制内容。

可以以多种方式在 XML 中使用自定义 Drawable 类：

1. 使用完全限定类名作为 XML 元素名称。对于此方法，自定义 drawable 类必须是公共顶级类。

   ``` xml
   <com.example.CustomDrawable xmlns:android="http://schemas.android.com/apk/res/android"/>
   ```

2. 使用 drawable 作为 XML 元素名称，并从类属性中指定完全限定的类名。此方法可用于公共顶层类和公共静态内部类。

   ``` xml
   <drawable xmlns:android="http://schemas.android.com/apk/res/android"
             class="com.example.TopDrawable$CustomDrawable"
             />
   ```


### Drawable.ConstantState

ConstantState 用于在不同的 Drawable 之间保存共同的恒定状态和数据。例如：从同一个资源创建的 `BitmapDrawables` 将共享保存在它们的 ConstantState 中的同一个单独的 Bitmap。

`newDrawable(Resources)` 可以作为一个工厂类被用来创建新的 Drawable 从当前的 ConstantState。

使用 `getConstantSatte()` 从一个 Drawable中取回 ConstantState。

在 Drawable 中调用 `mutate()`  通常应该为这个 Drawable 创建一个新的 ConstantState。

#### 自定义 Drawable 局限性

* 动态改变 Drawable 绘制的尺寸属性(比如，绘制的变局或者其中文字的大小)，需要调用设置了 Drawable 的对象(一般为 View )去重新测量尺寸。`getBounds` 方法可能返回不同的尺寸，但真实尺寸可能不会改变。

### 自定义 Drawable 实现抽象方法

``` java
//返回 Drawable 的真实尺寸。用于设置了 Drawable 的对象在测量时，考虑的合适尺寸
	@Override
    public int getIntrinsicWidth() {
        return mIntrinsicWidth;
    }

    @Override
    public int getIntrinsicHeight() {
        return mIntrinsicHeight;
    }

//返回当前 Drawable 为透明／不透明
public abstract @PixelFormat.Opacity int getOpacity();
```

