[文档地址](https://developer.android.com/topic/performance/rendering/profile-gpu.html#sam)

[文档地址](https://developer.android.com/studio/profile/dev-options-rendering.html#visual-output)

### 简介

GPU呈现模式分析( Profile GPU Rendering )可以给你快速可视化表示渲染 UI 窗口的每一帧所需的时间（每栈 16 毫秒为基准）。

移动设备运行在 Android 4.1 (API 16 ) 以上，而且开启了开发者模式。

> 注意：虽然这个工具被命名为 GPU呈现模式分析 (Profile GPU Rendering)，但所有被监视的进程实际上都在 CPU 中发生。渲染发生在提交命令到 GPU，GPU 同步渲染在屏幕。在某些情况下，GPU 有太多任务需要处理，CPU 将在可以提交新的命令之前一直等待。但这种情况发生时，你将看到更多的 **橙色** 和 **红色** ，命令提交将被阻塞，直到 GPU 命令队列上有更多的空间。

开启方式

![开启方式](https://developer.android.com/images/tools/performance/profile-gpu-rendering/gettingstarted_image001.png)

#### GPU分析器的视觉呈现

* 对于每个可见应用，工具会显示图形界面
* 水平轴显示时间的推移，垂直轴显示每帧所需的时间。(以毫秒为单位)
* 当你与应用交互时，垂直条出现在你的屏幕上，从左到右出现，随着时间推移，绘制出每帧的性能
* 每一个垂直条表示一帧的渲染。垂直条越高，渲染的时间越长
* 绿线标记 16 毫秒。每当有一帧越过绿线，就表示你的应用丢失了一帧，你的用户可能感觉到卡顿。

![界面](https://developer.android.com/images/tools/performance/profile-gpu-rendering/gettingstarted_image002.png)

### 图形分析

![图形](https://developer.android.com/topic/performance/images/bars.png)



![颜色](https://developer.android.com/topic/performance/images/s-profiler-legend.png)

* Input Handling

  管道的输入处理阶段测量应用花费多长时间处理输入事件。此度量指示应用花费多长时间执行由于输入事件回调而调用的代码。

  **当这个片段很大时**

  该区域中的高值通常是在输入处理程序事件回调中发生事件太多或者太复杂。因为这些回调总是发生在主线程，所有解决问题的方法就是直接优化处理或者将处理移动到不同的线程。

  值得注意的是 `RecyclerView` 滚动 也可以发生在这一阶段，当它消费触摸事件时，将立即开始滚动。因此，它可以 **inflate** 或 **populate** 新的 Item 视图。因为这个原因，重要的是尽可能的快速完成这一操作。

* Animation

  动画阶段显示了**评估所有在该帧运行的动画**的时长。常见的动画，比如：`ObjectAnimator`，`ViewPropertyAnimator`等等。

  **当这个片段很大时**

  该区域中的高值通常是由于动画某些属性的更改而导致。比如，在 `RecyclerView` 或 `ListView` 滚动时的 Fling 动画，会导致大量的视图 `inflation` 和 `population`。

* Measurement/Layout

  为了让 Android 在屏幕上绘制视图，它会在 `view hierarchy` 中的布局和视图执行两个特定操作。

  第一，系统测量视图，每个视图和布局都有特定的数据来描述这个对象在屏幕上的尺寸。一些视图可以有特定的尺寸；其他的具有适应父布局容器的尺寸。

  第二，系统布局视图，一旦计算好子视图的尺寸后，系统就可以继续对视图在屏幕上进行布局，调整尺寸和确定位置。

  系统不仅要对需要绘制的视图进行测量和布局，还需要对这些视图的父层级视图，直到根视图为止。

  **当这个片段很大时**

  如果你的应用在这个区域花费了大量的时间，它通常是因为有大量的视图需要处理，或者错误的 `view hierarchy` 导致的 `double  taxation` 问题。在任何一种情况下，优化 `view hierarchy` 提高性能。

  加在 `onLayout(boolean, int,int,int,int)` 和 `onMeasure(int,int)` 的代码也会引起性能问题。[Traceview](https://developer.android.com/studio/profile/traceview.html) 和 [Systrace](https://developer.android.com/studio/profile/systrace.html) 可以帮助你检查调用堆栈，以确定代码可能存在的问题。

* Drawing

  绘制阶段转换视图的渲染操作，比如转换绘制一个文本的背景或者绘制文本，变成原生绘制命令。系统捕获这些绘制命令到 `display list` 。

  绘制条记录将屏幕上所有需要更新的视图的绘制命令捕获到 `display list` 所需的时间。这些时间包括在应用中所有添加的 UI 对象的代码。例如可能是在 `onDraw()` ，`dispatchDraw()` 和 Drawable 子类中的 `draw()` 方法等等。

  **当这个片段很大时**

  简单来说，你可以理解为所有无效视图运行调用 `onDraw()` 的时间，这也包括所有花费在向子视图或者可能存在可绘制对象分发绘制命令的时间。因此，当你看到绘制区域条变得尖，原因可能是一堆视图突然变成无效，失效使得它需要重新生成视图的 `display list` 。另外，时间过长也可能是因为一些自定义控件在它们的 `onDraw()` 方法中存在极其复杂的业务逻辑。

* Sync/Upload

  同步和上传指标表示在当前帧中将位图对象从 CPU 内存传输到 GPU 内存所需的时间。

  作为不同的处理器，CPU 和 GPU 拥有不同的 RAM 地址用来处理。当你在 Android 中绘制一个位图时，系统会在 GPU 可以将它渲染到屏幕之前将它传输到 GPU 的内存中。然后，GPU 会缓存这个位图，所以系统不需要再次传输这个数据直到将它从缓存中清除。

  **当这个片段很大时**

  当前帧的所有资源需要存储在 GPU 内存中，才能用于绘制帧。这意味着该区域的高值可能存在大量的小资源加载或少数量却非常大的资源。一个常见的案例是当应用显示接近屏幕大小的单个位图时，另外一种是当应用显示大量缩略图时。

  要缩小这个栏的值，你可以采用以下技术：

  * 确保你的位图分辨率不会大于其用来显示的大小。例如，你的应用应避免将 1024 x 1024 的图像用来显示为 48 x 48 的图像。
  * 使用 `prepareToDraw()` 在下一个同步阶段之前异步预先映射位图。

* Issuing Commands

  发送命令阶段表示发布将 `display list` 绘制到屏幕所需的所有命令的时间。

  为了使系统将 `display list` 绘制到屏幕，它向 GPU 发送必要的命令。通常，它通过 OpenGL ES API 执行此操作。

  该过程需要一些时间，因为系统在将命令发送到 GPU 之前对每个命令执行最终的转换和剪切。然后在 GPU side 产生额外的开销，GPU side 计算最终的命令。这些命令包括最终的转换和额外的剪切。

  **当这个片段很大时**

  在这个阶段花费的时间是系统在给定的帧中呈现的 `display list` 的复杂性和数量的直接度量。比如，使用许多绘制操作，特别是在每个绘制原始资源的固定成本很小的情况下，这个时间可能会膨胀。例如：

  ``` java
  for(int i = 0; i < 1000; i++)
    canvas.drawPoint();
  ```

  比这个操作更昂贵：

  ``` java
  canvas.drawPoints(mThousandPointArray);
  ```

  发送命令和实际绘制 `display list` 之间并不总是有 1:1 的相关性。`issuing command` 是捕获将绘制命令（`display list`）发送到 GPU 所花费的时间，`draw` 操作是表示将已发出命令捕获到 `display list` 中所花费的时间。

  出现这种差异是因为 `display list` 在可能的情况下被系统缓存。因此，在一些情况下，滚动，变换或动画需要系统重新发送 `display list` ，而不必实际重新渲染它。因此，你可能看到一个高的 `issuing comman` 栏，而没有看到一个高的 `draw` 栏。

* Processing/Swapping Buffers

  一旦 Android 结束提交所有 `display list` 到 GPU 中，系统将发送一个最终的命令去告诉图形驱动程序当前帧已完成了。此时，图形驱动程序可以更新的图像呈现给屏幕。

  **当这个片段很大时**

  了解 GPU 运行与 CPU 并行的工作很重要。Android 系统发送绘制命令到 GPU，然后继续下一个任务。GPU 从队列中读取这些绘制命令并且处理它们。

  在 CPU 发送命令比 GPU 消费命令更快的情况，处理器之间的通信队列可以变满。发生这种情况时，CPU 会阻塞，并等待队列中有空位放置下一个命令。这种全队列状态通常在交换缓冲区阶段出现，因为在这一点上，提交了整帧的命令。减轻这个问题的关键在于减少在 GPU 上发生工作的复杂性。类似在 `Issue Comands` 阶段。

* Miscellanous

  除了渲染系统执行其工作所需的时间之外，还有一组额外的工作发生在主线程，与渲染无关。这个工作消耗的时间被报告为杂项时间。杂项时间通常表示在两个连续渲染帧之间的 UI 线程上可能发生的工作。

  **当这个片段很大时**

  如果此值很高，那么应用很可能具有回调，意图，或者其他应该在其他线程发生的工作。使用比如像[Method Tracing](https://developer.android.com/studio/profile/traceview.html) 或 [Systace](https://developer.android.com/studio/profile/systrace.html) 可以提供在主线程上运行的任务的可见性。这信息可以帮助你针对性能改进。