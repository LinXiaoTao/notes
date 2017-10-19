[参考](http://blog.csdn.net/iispring/article/details/50472485)

[参考](http://blog.csdn.net/cquwentao/article/details/51423371)

### PorterDuffXfermode

在用Android中的Canvas进行绘图时，可以通过使用PorterDuffXfermode将所绘制的图形的像素与Canvas中对应位置的像素按照一定规则进行混合，形成新的像素值，从而更新Canvas中最终的像素颜色值，这样会创建很多有趣的效果。当使用PorterDuffXfermode时，需要将将其作为参数传给`Paint.setXfermode(Xfermode xfermode)`方法，这样在用该画笔paint进行绘图时，Android就会使用传入的PorterDuffXfermode，如果不想再使用Xfermode，那么可以执行`Paint.setXfermode(null)`。

### Layer(图层)

`saveLayer`可以为canvas创建一个新的透明图层，在新的图层上绘制，并不会直接绘制到屏幕上，而会在restore之后，绘制到上一个图层或者屏幕上（如果没有上一个图层）。为什么会需要一个新的图层，例如在处理xfermode的时候，原canvas上的图（包括背景）会影响src和dst的合成，这个时候，使用一个新的透明图层是一个很好的选择。

又例如需要当前绘制的图形都带有一定的透明度，那么使用`saveLayerAlpha`创建一个带有透明度的图层，也是一个方便的选择。

### saveLayer和save

`save`的功能与`saveLayer`的功能类似，但`save`是将当前在Canvas上的操作根据**saveFlags**保存到私有堆栈上。

### SaveFlags

首先需要知道的是，使用flag的方法除了saveFlayer还有save方法，他们都可以使用flag来指定需要保存的信息，在恢复时候起作用。那么来看看flag所对应的意义：

| Flag                       | 意义                                       | 适用方法           |
| -------------------------- | ---------------------------------------- | -------------- |
| MATRIX_SAVE_FLAG           | 保存位图的matrix矩阵                            | save;saveFayer |
| CLIP_SAVE_FLAG             | 保存位图的裁剪大小信息                              | save;saveFayer |
| HAS_ALPHA_LAYER_SAVE_FLAG  | 保存位图的透明通道信息，和下面的标识冲突，都设置时以下面的标志为准        | saveFayer      |
| FULL_COLOR_LAYER_SAVE_FLAG | 保存位图所有颜色信息(每个通道8位)（和上一图层合并时，清空上一图层的重叠区域，保留该图层的颜色） | saveFayer      |
| CLIP_TO_LAYER_SAVE_FLAG    | 创建图层时，会把canvas（所有图层）裁剪到参数指定的范围，如果省略这个flag将导致图层开销巨大（实际上图层没有裁剪，与原图层一样大） | saveFayer      |
| ALL_SAVE_FLAG              | 所有信息                                     | save;saveFayer |

