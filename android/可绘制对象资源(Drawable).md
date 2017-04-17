### 可绘制对象资源

[文档地址](https://developer.android.google.cn/guide/topics/resources/drawable-resource.html)

可绘制对象资源是一般概念，是指可在屏幕上绘制的图形，以及可以使用 `getDrawable(int)` 等 API 检索或者应用到具有 `android:drawable` 和 `android:icon` 等属性的其他 XML 资源的图形。共有多种不同类型的可绘制对象：

* 位图文件

  位图图形文件（`.png`、`.jpg` 或 `.gif`）。创建 `BitmapDrawable`。

* 九宫格文件

  具有可拉伸区域的 PNG 文件，允许根据内容调整图像大小 (`.9.png`)。创建 `NinePatchDrawable`。

* 图层列表

  管理其他可绘制对象阵列的可绘制对象。它们按阵列顺序绘制，因此索引最大的元素绘制在顶部。创建 `LayerDrawable`。

* 状态列表

  此 XML 文件为不同状态引用不同位图图形（例如，按下按钮时使用不同的图像）。创建 `StateListDrawable`。

* 级别列表

  此 XML 文件用于定义管理大量备选可绘制对象的可绘制对象，每个可绘制对象都分配有最大的备选数量。创建 `LevelListDrawable`。

* 转换可绘制对象

  此 XML 文件用于定义可在两种可绘制对象资源之间交错淡出的可绘制对象。创建 `TransitionDrawable`。

* 插入可绘制对象

  此 XML 文件用于定义以指定距离插入其他可绘制对象的可绘制对象。当视图需要小于视图实际边界的背景可绘制对象时，此类可绘制对象很有用。

* 裁剪可绘制对象

  此 XML 文件用于定义对其他可绘制对象进行裁剪（根据其当前级别值）的可绘制对象。创建 `ClipDrawable`。

* 缩放可绘制对象

  此 XML 文件用于定义更改其他可绘制对象大小（根据其当前级别值）的可绘制对象。创建 `ScaleDrawable`

* 形状可绘制对象

  此 XML 文件用于定义几何形状（包括颜色和渐变）。创建 ShapeDrawable。

* 帧动画

  AnimationDrawable

* 颜色可绘制对象

  [颜色资源](https://developer.android.google.cn/guide/topics/resources/more-resources.html?hl=zh-cn#Color)也可用作 XML 中的可绘制对象。例如，在创建[状态列表可绘制对象](https://developer.android.google.cn/guide/topics/resources/drawable-resource.html?hl=zh-cn#StateList)时，可以引用 `android:drawable` 属性的颜色资源 (`android:drawable="@color/green"`)。ColorDrawable

#### 位图

位图图像。Android 支持以下三种格式的位图文件：`.png`（首选）、`.jpg`（可接受）、`.gif`（不建议）。

您可以使用文件名作为资源 ID 直接引用位图文件，也可以在 XML 中创建别名资源 ID。

**注**：在构建过程中，可通过 `aapt` 工具自动优化位图文件，对图像进行无损压缩。因此请注意，此目录中的图像二进制文件在构建时可能会发生变化。如果您计划将图像解读为比特流以将其转换为位图，请改为将图像放在 `res/raw/` 文件夹中，在那里它们不会进行优化。

* 位图文件

  位图文件是 `.png`、`.jpg` 或 `.gif` 文件。当您将这些文件保存到 `res/drawable/` 目录中时，Android 将为它们创建 `BitmapDrawable` 资源。

  在xml文件中使用：

  ``` xml
  <ImageView
      android:layout_height="wrap_content"
      android:layout_width="wrap_content"
      android:src="@drawable/myimage" />
  ```

  在代码文件中使用：

  ``` java
  Resources res = getResources();
  Drawable drawable = res.getDrawable(R.drawable.myimage);
  ```

* XML位图

  XML 位图是在 XML 中定义的资源，指向位图文件。实际上是原始位图文件的别名。XML 可以指定位图的其他属性，例如抖动和层叠。

  ``` xml
  <?xml version="1.0" encoding="utf-8"?>
  <bitmap
      xmlns:android="http://schemas.android.com/apk/res/android"
      android:src="@[package:]drawable/drawable_resource" 可绘制对象资源。  
      android:antialias=["true" | "false"] 启用或停用抗锯齿。
      android:dither=["true" | "false"] 当位图的像素配置与屏幕不同时（例如：ARGB 8888 位图和 RGB 565 屏幕），启用或停用位图抖动。

      android:filter=["true" | "false"] 启用或停用位图过滤。当位图收缩或拉伸以使其外观平滑时使用过滤。
      android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                        "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                        "center" | "fill" | "clip_vertical" | "clip_horizontal"]         定义位图的重力。重力指示当位图小于容器时，可绘制对象在其容器中放置的位置。


      android:mipMap=["true" | "false"] 启用或停用 mipmap 提示。
      android:tileMode=["disabled" | "clamp" | "repeat" | "mirror"] /> 定义平铺模式。当平铺模式启用时，位图会重复。重力在平铺模式启用时将被忽略。
  ```

  ​

