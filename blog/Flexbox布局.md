**这篇文章主要参考了[官方文档](https://github.com/google/flexbox-layout)和阮老师的[Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)**

[github地址](https://github.com/LinXiaoTao/FlexboxExample)

### 简介

最近Google开源了一个叫[flex-box](https://github.com/google/flexbox-layout)的库，它的思路是参照的**CSS的Flex布局**设计的，所以属性基本都是和CSS上的Flex布局保持一致，但因为是两个不同的平台，所以减少了几个不适用于Android的属性，新增了几个属性，具体我们下面会说到。

### 关于CSS的Flex布局

在了解Google开源的**flex-box**，我觉得有必要先了解下**CSS的Flex布局**，这有助于我们理解那些属性定义。关于这方面的知识，可以阅读这篇[博客](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)。下面我们只是简要的了解下，具体属性我们放到讲解Google的**flex-box**中去说明。

```
//这段文字说明，来源于上面说到的博客
采用Flex布局的元素，称为Flex容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为Flex项目（flex item），简称"项目"。
容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；交叉轴的开始位置叫做cross start，结束位置叫做cross end。
项目默认沿主轴排列。单个项目占据的主轴空间叫做main size，占据的交叉轴空间叫做cross size。
```

![img](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071004.png)

### 关于Google的flexbox-layout

**注意：以下文中用到的flexbox-layout指的是Google开源的库，Flex指的是CSS中的Flex布局。**

首先我们需要知道的是，flexbox-layout的基本概念和Flex是一样的，最外面的这层布局称为**容器**，在flexbox-layout指的是`com.google.android.flexbox.FlexboxLayout`，里面的子元素称为**项目**。容器默认存在两条轴，其中水平(这里是就上面的概念图而言，因为主轴的方向是可以指定的)的这条轴称为**主轴**(个人理解，可以存在多条，具体原因后面讲)，垂直的这条称为**交叉轴**。其他属性比如`start`，`end`等等也是一样的，具体可以看上面的基本概念图。

下面我们着重讲下flexbox-layout支持的属性，因为它既支持容器属性，也支持项目属性，所以我们分成两个篇幅来说明。

**注意：以下属性设置的具体效果请参照说明文档和Demo，[github地址](https://github.com/google/flexbox-layout)**

#### 容器属性

* **flexDirection**

  这个属性用于确定**主轴方向**，可选的值有：

  1. row 水平方向，起点在左边。默认值
  2. row_revers 水平方向，起点在右边。
  3. column 垂直方向，起点在上边。
  4. column_reverse 垂直方向，起点在下边。

  ![img](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071005.png)

* **flexWrap**

  这个属性控制容器是单行还是自动换行，以及主轴的方向，可选的值有：

  1. nowrap 单行。默认值
  2. wrap 自动换行，第一行在上方
  3. wrap-reverse 自动换行，第一行在下方

  关于`wrap`和`wrap-reverse`的区别：

  ![img](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071008.jpg)

  ![img](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071009.jpg)

* **justifyContent**

  这个属性控制项目在主轴上的对齐方式，可选的值有：

  **注意：具体的对齐方式与主轴的方向有关，下面的说明是基于默认值**

  1. flex_start 左对齐，默认值
  2. flex_end 右对齐
  3. center 居中对齐
  4. space_between 两端对齐，项目之间的间隔相等
  5. space_around  每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍

  关于`space_between `和`space_around `的样式如下：

  ![img](https://github.com/LinXiaoTao/notes/blob/master/blog/img/flexbox-layout_space_between.png)

  ![img](https://github.com/LinXiaoTao/notes/blob/master/blog/img/flexbox-layout_space_around.png)

* **alignItems**

  这个属性定义项目在交叉轴上的对齐方式，可选的值有：

  **注意：具体的对齐方式与主轴的方向有关**

  1. stretch 占满整个容器高度，默认值
  2. flex_start 交叉轴的起点对齐
  3. flex_end 交叉轴的终点对齐
  4. center 居中对齐
  5. baseline 项目的第一行文字的基线对齐

  关于`stretch  `和`baseline  `的样式如下：

  ![img](https://github.com/LinXiaoTao/notes/blob/master/blog/img/flexbox-layout_stretch.png)

  ![img](https://github.com/LinXiaoTao/notes/blob/master/blog/img/flexbox-layout_baseline.png)

* **alignContent**

  这个属性控制容器的多条主轴的对齐方式，这也是我们前面提到的**主轴可以有多条，当`flexWrap:wrap|wrapreverse`**。可选的值有：

  **注意：只有一条主轴，该属性不起作用**

  1. stretch 占满整个交叉轴，默认值
  2. flex_start 与交叉轴的起点对齐
  3. flex_end 与交叉轴的终点对齐
  4. center 与交叉轴的中心对齐
  5. space_between 与交叉轴两端对齐，轴线之间的间隔平均分布
  6. space_around 每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍

  这里需要注意区别下和`alignItems`的区别，`alignContent`是多条主轴基于交叉轴的对齐方式，而后者是项目基于交叉轴的对齐方式，一个是轴，一个是项目。

* **showDividerHorizontal**

* **dividerDrawableHorizontal**

* **showDividerVertical**

* **dividerDrawableVertical**

* **showDivider**

* **dividerDrawable** 

  这几个属性都是跟分隔符有关，具体用法可以看文档。

#### 项目属性

* **layout_order**(integer)

  这个属性可以改变项目的排列顺序，默认值为`1`。

* **layout_flexGrow**(float)

  这个属性定义项目基于当前行的放大比例，默认值为`0`。这个属性类似`LinearLayout`中的`layout_weight`属性，如果当前行只有一个项目的`layout_flexGrow`为正值，则该项目将占满当前行剩余的空间，下面的效果是三个相同大小的项目，其中一个`layout_flexGrow`为正值的效果：

  ![img](https://github.com/LinXiaoTao/notes/blob/master/blog/img/flexbox-layout_layout_flexGrow1.png)

  如果存在多个`layout_flexGrow`为正值的情况，则这几个项目则会按设置的值为比例占用当前行剩余的空间，下面的效果是三个相同大小的项目，其中两个项目设置`layout_flexGrow:1`的效果：

  ![img](https://github.com/LinXiaoTao/notes/blob/master/blog/img/flexbox-layout_layout_flexGrow2.png)

* **layout_flexShrink** (float)

  这个属性定义了项目的缩小比例，默认为`1`，即如果当前行空间不足，该项目缩小的方式。

  如果所有项目的`layout_flexShrink`属性都为1，当空间不足时，都将等比例缩小。如果一个项目的`layout_flexShrink`属性为0，其他项目都为1，则空间不足时，前者不缩小。

  **个人理解，注意：要设置`flexWrap:nowrap`为单行**

* **layout_alignSelf**

  该属性允许单个项目与其他项目不一样的基于交叉轴的对齐方式。默认值为`auto`，即按照容器的`alignItems`属性，设置其他值，则会覆盖容器的值，可选的值：

  1. auto 默认值
  2. flex_start 
  3. flex_end
  4. center
  5. baseline
  6. stretch

* **layout_flexBasisPercent** (fraction)

  这个属性设置项目长度相对于容器的百分比，如果设置了这个值，则从`layout_width`或`layout_height`指定的长度会被覆盖，需要注意的是，这个属性只在容器长度确定的情况下有效，即`MeasureSpec.EXACTLY`。默认值为`-1`，表示不设置。

* **layout_minWidth** / **layout_minHeight** (dimension)

* **layout_maxWidth** / **layout_maxHeight** (dimension)

  这些属性设置对项目的最大最小限制

* **layout_wrapBefore** (boolean)

  这个属性默认值为`false`，如果设置为`true`，则该项目将强制成为当前行的第一个项目，会忽略`flex_wrap:nowrap`设置。

### 应用实例

1. 底部按钮

   ![img](https://github.com/LinXiaoTao/notes/blob/master/blog/img/flexbox-layout_fixbottom.png)

2. 流式布局

   ![img](https://github.com/LinXiaoTao/notes/blob/master/blog/img/flexbox-layout_flowlayout.png)

TODO

