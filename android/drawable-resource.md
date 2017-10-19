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

  位图文件是 `.png`、`.jpg` 或 `.gif` 文件。

  文件位置：`res/drawable/xx.png` 

  编译的资源数据类型：`BitmapDrawable`

  资源引用：

  1. 在 Java 中：`R.drawable.filename`
  2. 在 xml  中：`@[package:]drawable/filename`

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

  > 注：可以将 `<bitmap>` 元素用作 `<item>` 元素的子项

  文件位置：`res/drawable/xx.xml`

  编译的资源数据类型：`BitmapDrawable`

  资源引用：

  1. 在 Java 中：`R.drawable.filename`
  2. 在 xml  中：`@[package:]drawable/filename

   例子：

  ``` xml
  <?xml version="1.0" encoding="utf-8"?>
  <bitmap
      xmlns:android="http://schemas.android.com/apk/res/android"
      android:src="@[package:]drawable/drawable_resource"
      android:antialias=["true" | "false"]//抗锯齿
      android:dither=["true" | "false"]
      android:filter=["true" | "false"]
      android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                        "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                        "center" | "fill" | "clip_vertical" | "clip_horizontal"]
      android:mipMap=["true" | "false"]
      android:tileMode=["disabled" | "clamp" | "repeat" | "mirror"] />
  ```

  在代码中使用：

  ``` java
  Drawable drawable = ResourcesCompat.getDrawable(getResources(), R.drawable.bitmap_test, getTheme());
          mIamgeView2.setImageDrawable(drawable);
  ```


#### 九宫图

`NinePatch` 是一种 PNG 图像，在其中可定义当视图中的内容超出正常图像边界时 Android 缩放的可拉伸区域。此类图像通常指定为至少有一个尺寸设置为 `"wrap_content"` 的视图的背景，而且当视图扩展以适应内容时，九宫格图像也会扩展以匹配视图的大小。

* 九宫图文件

  文件位置：`res/drawable/xx.9.png`

  编译的资源数据类型：`NinePatchDrawable`

  资源引用：

  1. 在 Java 中：`R.drawable.filename`
  2. 在 xml  中：`@[package:]drawable/filename`

* XML九宫图

  XML 九宫格是在 XML 中定义的资源，指向九宫格文件。XML 可以为图像指定抖动。

  文件位置：`res/drawable/filename.xml`

  编译的资源数据类型：`NinePatchDrawable`

  资源引用：

  1. 在 Java 中：`R.drawable.filename`
  2. 在 xml  中：`@[package:]drawable/filename

  语法：

  ``` xml
  <?xml version="1.0" encoding="utf-8"?>
  <nine-patch
      xmlns:android="http://schemas.android.com/apk/res/android"
      android:src="@[package:]drawable/drawable_resource"
      android:dither=["true" | "false"] />//当位图的像素配置与屏幕不同时（例如：ARGB 8888 位图和 RGB 565 屏幕），启用或停用位图抖动。
  ```

#### 图层列表

`LayerDrawable` 是管理其他可绘制对象阵列的可绘制对象。列表中的每个可绘制对象按照列表的顺序绘制，列表中的最后一个可绘制对象绘于顶部。

每个可绘制对象由单一 `<layer-list>` 元素内的 `<item>` 元素表示。

文件位置：`res/drawable/filename.xml`

编译的资源数据类型：`LayerDrawable`

资源引用：

1. 在 Java 中：`R.drawable.filename`
2. 在 xml  中：`@[package:]drawable/filename

语法：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:id="@[+][package:]id/resource_name"
        android:top="dimension"
        android:right="dimension"
        android:bottom="dimension"
        android:left="dimension" />
</layer-list>
```

默认情况下，所有可绘制项都会缩放以适应包含视图的大小。因此，将图像放在图层列表中的不同位置可能会增大视图的大小，并且有些图像会相应地缩放。为避免缩放列表中的项目，请在 `<item>` 元素内使用 `<bitmap>` 元素指定可绘制对象，并且对某些不缩放的项目（例如 `"center"`）定义重力。例如，以下 `<item>` 定义缩放以适应其容器视图的项目：

``` xml
<item android:drawable="@drawable/image" />
```

为避免缩放，以下示例使用重力居中的 `<bitmap>` 元素：

``` xml
<item>
  <bitmap android:src="@drawable/image"
          android:gravity="center" />
</item>
```

#### 状态列表

`StateListDrawable` 是在 XML 中定义的可绘制对象，它根据对象的状态，使用多个不同的图像来表示同一个图形。

您可以在 XML 文件中描述状态列表。每个图形由单一 `<selector>` 元素内的 `<item>` 元素表示。每个 `<item>` 均使用各种属性来描述应用作可绘制对象的图形的状态。

在每个状态变更期间，将从上到下遍历状态列表，并使用第一个与当前状态匹配的项目 —此选择并非**基于“最佳匹配”，而是选择符合状态最低条件的第一个项目。

文件位置：`res/drawable/filename.xml`

编译的资源数据类型：`StateListDrawable`

资源引用：

1. 在 Java 中：`R.drawable.filename`
2. 在 xml  中：`@[package:]drawable/filename

语法：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android"
    android:constantSize=["true" | "false"]
    android:dither=["true" | "false"]
    android:variablePadding=["true" | "false"] >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:state_pressed=["true" | "false"]
        android:state_focused=["true" | "false"]
        android:state_hovered=["true" | "false"]
        android:state_selected=["true" | "false"]
        android:state_checkable=["true" | "false"]
        android:state_checked=["true" | "false"]
        android:state_enabled=["true" | "false"]
        android:state_activated=["true" | "false"]
        android:state_window_focused=["true" | "false"] />
</selector>
```

> **注**：请记住，Android 将应用状态列表中第一个与对象当前状态匹配的项目。因此，如果列表中的第一个项目不含上述任何状态属性，则每次都会应用它，这就是默认值应始终放在最后的原因

#### 级别列表

管理大量备选可绘制对象的可绘制对象，每个可绘制对象都分配有最大的备选数量。使用 `setLevel()` 设置可绘制对象的级别值会加载级别列表中 `android:maxLevel` 值大于或等于传递到方法的值的可绘制对象资源。

文件位置：`res/drawable/filename.xml`

编译的资源数据类型：`LevelListDrawable`

资源引用：

1. 在 Java 中：`R.drawable.filename`
2. 在 xml  中：`@[package:]drawable/filename

语法：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<level-list
    xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@drawable/drawable_resource"
        android:maxLevel="integer"
        android:minLevel="integer" />
</level-list>
```

在此项目应用到 `View` 后，可通过 `setLevel()` 或 `setImageLevel()` 更改级别。

#### 转换可绘制对象

`TransitionDrawable` 是可在两种可绘制对象资源之间交错淡出的可绘制对象。

每个可绘制对象由单一 `<transition>` 元素内的 `<item>` 元素表示。不支持超过两个项目。要向前转换，请调用 `startTransition()`。要向后转换，则调用 `reverseTransition()`。

文件位置：`res/drawable/filename.xml`

编译的资源数据类型：`TransitionDrawable`

资源引用：

1. 在 Java 中：`R.drawable.filename`
2. 在 xml  中：`@[package:]drawable/filename`

语法：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<transition
xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:id="@[+][package:]id/resource_name"
        android:top="dimension"
        android:right="dimension"
        android:bottom="dimension"
        android:left="dimension" />
</transition>
```

#### 裁剪可绘制对象

在 XML 文件中定义的对其他可绘制对象进行裁剪（根据其当前级别）的可绘制对象。您可以根据级别以及用于控制其在整个容器中位置的重力，来控制子可绘制对象的裁剪宽度和高度。通常用于实现进度栏之类的项目。

文件位置：`res/drawable/filename.xml`

编译的资源数据类型：`ClipDrawable`

资源引用：

1. 在 Java 中：`R.drawable.filename`
2. 在 xml  中：`@[package:]drawable/filename`

语法：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<clip
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/drawable_resource"
    android:clipOrientation=["horizontal" | "vertical"]
    android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                     "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                     "center" | "fill" | "clip_vertical" | "clip_horizontal"] />
```

> **注**：默认级别为 0，即完全裁剪，使图像不可见。当级别为 10,000 时，图像不会裁剪，而是完全可见。

#### 缩放可绘制对象

在 XML 文件中定义的更改其他可绘制对象大小（根据其当前级别）的可绘制对象。

文件位置：`res/drawable/filename.xml`

编译的资源数据类型：`ScaleDrawable`

资源引用：

1. 在 Java 中：`R.drawable.filename`
2. 在 xml  中：`@[package:]drawable/filename`

语法：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<scale
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/drawable_resource"
    android:scaleGravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                          "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                          "center" | "fill" | "clip_vertical" | "clip_horizontal"]
    android:scaleHeight="percentage"
    android:scaleWidth="percentage" />
```

| 属性                   | 备注                                       |
| -------------------- | ---------------------------------------- |
| android:scaleGravity | *关键字*。指定缩放后的重力位置。                        |
| android:scaleHeight  | *百分比*。缩放高度，表示为可绘制对象边界的百分比。值的格式为 XX%。例如：100%、12.5% 等。 |
| android:scaleWidth   | *百分比*。缩放宽度，表示为可绘制对象边界的百分比。               |

> 注：如果将 `ScaleDrawable` 设置给 `ImageView` ，那么在 xml 中设置的 `level` 会被 `ImageView` 的 `level` 覆盖。

#### 形状可绘制对象

这是在 XML 中定义的一般形状。

文件位置：`res/drawable/filename.xml`

编译的资源数据类型：`GradienDrawable`

资源引用：

1. 在 Java 中：`R.drawable.filename`
2. 在 xml  中：`@[package:]drawable/filename`

语法：

``` xm
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape=["rectangle" | "oval" | "line" | "ring"] >
    <corners
        android:radius="integer"
        android:topLeftRadius="integer"
        android:topRightRadius="integer"
        android:bottomLeftRadius="integer"
        android:bottomRightRadius="integer" />
    <gradient
        android:angle="integer"
        android:centerX="float"
        android:centerY="float"
        android:centerColor="integer"
        android:endColor="color"
        android:gradientRadius="integer"
        android:startColor="color"
        android:type=["linear" | "radial" | "sweep"]
        android:useLevel=["true" | "false"] />
    <padding
        android:left="integer"
        android:top="integer"
        android:right="integer"
        android:bottom="integer" />
    <size
        android:width="integer"
        android:height="integer" />
    <solid
        android:color="color" />
    <stroke
        android:width="integer"
        android:color="color"
        android:dashWidth="integer"
        android:dashGap="integer" />
</shape>
```

| 属性            | 备注                                       |
| ------------- | ---------------------------------------- |
| android:shape | *键字*。定义形状的类型。有效值为：` rectangle(矩形)`，`oval(椭圆)`，`line(水平线)`，`ring(环形)` |

> 注：当 `android:shape : line` ，此形状需要 `<stroke>` 元素定义线宽。

仅当 `android:shape:"ring"` 才使用以下属性：

| 属性                       | 备注                                       |
| ------------------------ | ---------------------------------------- |
| android:innerRadius      | *尺寸*。环内部（中间的孔）的半径，以尺寸值或[尺寸资源](https://developer.android.com/guide/topics/resources/more-resources.html?hl=zh-cn#Dimension)表示。 |
| android:innerRadiusRatio | *浮点型*。环内部的半径，以环宽度的比率表示。例如，如果 `android:innerRadiusRatio="5"`，则内半径等于环宽度除以 5。此值被 `android:innerRadius` 覆盖。默认值为 9。 |
| android:thickness        | *尺寸*。环的厚度，以尺寸值或[尺寸资源](https://developer.android.com/guide/topics/resources/more-resources.html?hl=zh-cn#Dimension)表示。 |
| android:thicknessRatio   | *浮点型*。环的厚度，表示为环宽度的比率。例如，如果 `android:thicknessRatio="2"`，则厚度等于环宽度除以 2。此值被 `android:innerRadius` 覆盖。默认值为 3。 |
| android:useLevel         | *布尔值*。如果这用作 `LevelListDrawable`，则此值为“true”。这通常应为“false”，否则形状不会显示。 |
| android:thickness        | *尺寸*。环的厚度，以尺寸值或[尺寸资源](https://developer.android.com/guide/topics/resources/more-resources.html?hl=zh-cn#Dimension)表示。 |

`<corners>` 为形状产生圆角。仅当形状为矩形时适用。

| 属性               | 备注                                       |
| ---------------- | ---------------------------------------- |
| `android:radius` | *尺寸*。所有角的半径，以尺寸值或[尺寸资源](https://developer.android.com/guide/topics/resources/more-resources.html?hl=zh-cn#Dimension)表示。 |

> **注**：（最初）必须为每个角提供大于 1 的角半径，否则无法产生圆角。如果希望特定角不要倒圆角，解决方法是使用 `android:radius`设置大于 1 的默认角半径，然后使用实际所需的值替换每个角，为不希望倒圆角的角提供零（“0dp”）。

`<gradient>` 指定形状的渐变颜色。

| 属性                     | 备注                                       |
| ---------------------- | ---------------------------------------- |
| `android:angle`        | *整型*。渐变的角度（度）。0 为从左到右，90 为从上到上。必须是 45 的倍数。默认值为 0。 |
| android:centerX        | *浮点型*。渐变中心的相对 X 轴位置 (0 - 1.0)。           |
| `android:centerY`      | *浮点型*。渐变中心的相对 Y 轴位置 (0 - 1.0)。           |
| android:centerColor    | *颜色*。起始颜色与结束颜色之间的可选颜色，以十六进制值或[颜色资源](https://developer.android.com/guide/topics/resources/more-resources.html?hl=zh-cn#Color)表示。 |
| android:endColor       | *颜色*。结束颜色，表示为十六进制值或[颜色资源](https://developer.android.com/guide/topics/resources/more-resources.html?hl=zh-cn#Color)。 |
| android:gradientRadius | *浮点型*。渐变的半径。仅在 `android:type="radial"` 时适用。 |
| android:startColor     | *颜色*。起始颜色，表示为十六进制值或[颜色资源](https://developer.android.com/guide/topics/resources/more-resources.html?hl=zh-cn#Color)。 |
| android:type           | *关键字*。要应用的渐变图案的类型。有效值为：`linear( 线性渐变。这是默认值)`，`radial( 径向渐变。起始颜色为中心颜色)`，`sweep(流线型渐变)` |
| android:useLevel       | *布尔值*。如果这用作 `LevelListDrawable`，则此值为“true”。 |

`<padding>` 要应用到包含视图元素的内边距（这会填充视图内容的位置，而非形状）

`<size>` 形状的大小。

`<solid>` 用于填充形状的纯色。

`<stroke>` 形状的笔划中线。