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



