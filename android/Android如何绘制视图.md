当 `Activity` 获取到焦点时，它将请求绘制布局。Android framework 将处理绘制程序，但 `Activity` 必须提供布局层次的根结点。

绘制开始于布局的根节点，要求测量 (measure) 和绘制 (draw) 布局树。绘制程序是通过遍历整个树，并渲染 `View` 中每个被标记为无效 (invalid) 的区域。每个 `ViewGroup` 负责请求它的每个子 `View` 进行绘制 (`draw()`)，每一个 `View` 负责绘制它自己。因为树势按顺序进行遍历，这意味着父 `View` 将在它们的子 `View` 之前被绘制(处于更后方)。同级的将按照它们在树中的出现顺序绘制。

> 不在无效区域中的 `View` 不会被绘制，同时 framework 会为你管理处于后台的 `View` 的绘制，可以通过调用 `invalidate()` 强制 `View` 进行绘制。

绘制布局有两个过程：测量过程和布局过程。测量过程时在 `measure(int,int)` 中实施的，并且这是一个自上到下遍历的过程。在这个递归期间，每个 `View` 会将尺寸规格沿着树向下传递。在绘制过程的最后，所有的 `View` 都保存各自的测量尺寸。第二个过程发生在 `layout(int,int,int,int)` ，同时也是自上向下。在这个过程中，所有的父 `View` 负责根据在测量过程中的尺寸来定位它的所有子 `View` 。当 `View` 的 `measure()` 方法返回时，它的 `getMeasuredWidth()` 和 `getMeasuredHeight()` 的值必须被设置，以及所有 `View` 的后代。一个 `View` 的测量宽度和测量高度必须遵守它们的父 `View` 强加的约束。这保证了在测量结束时，所有的父 `View` 都能接受它们子 `View` 的尺寸。父 `View` 可能会多次调用子 `View` 的 `measure()` 方法。例如，父 `View` 可能会用未指定的尺寸测量每个子 `View` 一次，以了解它们想要的尺寸，如果所有子 `View` 未指定约束的尺寸总和过大或者过小，就会用真实的尺寸再次调用 `measure()` (即如果子 `View` 不接受它们之间所分配的空间大小，父 `View` 将干预，并在第二个过程中设定规则)。

> 当认为无法适应当前界限时，要启动 `layout` ，请通过 `View` 本身调用 `requestLayout`。

测量过程使用两个类传递尺寸。`View` 使用 `ViewGroup.LayoutParams` 来告知它们的父 `View` ，它们想要如何测量和定位自己。基类 `ViewGroup.LayoutParams` 描述 `View` 想要多大的宽度和高度。对于每个尺寸，它可以指定如下指：

* 具体数值
* MATCH_PARENT` ，意味 `View` 想要和父 `View` 一样大小(减去内边距 padding )。` 
* `WRAP_CONTENT` ，意味 `View` 想要适应其内容的大小(加上内边距 padding )。

不同的 `ViewGroup` 有对应的 `ViewGroup.LayoutParams` 的子类。例如，`RelativeLayout` 有它对于的 `ViewGroup.LayoutParams` 的子类，包含了可以设置子 `View` 水平和垂直居中。

`MeasureSpecs` 用于自树的父节点到子节点向下传递需求。一个 `MeasureSpec` 可以为以下指：

* `UNSPECIFIED`：由父 `View` 来决定子 `View` 所需的尺寸。例如，一个 `LinearLayout` 可能在调用子 `View` 的 `measure()` 时，设置高度为 `UNSPECIFIED` ，宽度为 240 ，来找寻子 `View` 在给予 240 像素宽度，想要多高。
* `EXACTLY` ：由父 `View` 来为子 `View` 强制指定一个实际的尺寸，子 `View` 必须使用这个尺寸 ，并保证它的所有后代都适应这个尺寸。
* `AT MOST`：由父 `View` 来为子 `View` 强制指定最大尺寸。子 `View` 必须保证它和它的后代都适应这个尺寸。



