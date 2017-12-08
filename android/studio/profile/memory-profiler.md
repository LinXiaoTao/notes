# Memory Profiler



## 作用

* 在时间轴中寻找可能导致性能问题的不良内存分配模式
  * Dump Java heap，以查看在任何给定时间哪些对象正在使用内存。
  * 在正常和极端的用户交互过程中录制内存分配，以确定代码在短时间内分配太多对象或泄露内存对象的位置。

> heap dump 是一个二进制文件（.hprof），它保存了某一时刻 JVM 堆中对象的使用情况。



## 概况

![Memory Profiler](https://developer.android.com/studio/images/profile/memory-profiler-callouts_2x.png)

Memory Profiler 默认包含以下：（序号对于图中编号）

1. 强制 GC 操作。
2. heap dump。
3. 录制内存分配。只能在 7.1 或 7.1 以下的设备上使用。
4. 缩放时间轴。
5. 定位实时内存情况。
6. 时间轴显示 activity 状态，用户输入，屏幕旋转事件。
7. 时间轴使用内存情况：
   * 每个内存类别使用多少内存的堆栈图，如左侧的 y 轴和顶部的颜色键所示。
   * 虚线表示分配对象的数量，如右边的y轴所示。
   * 图标表示每个 GC 事件。



### 内存是如何计算的

![memory counted](https://developer.android.com/studio/images/profile/memory-profiler-counts_2x.png)



上图内存计数是基于当前应用的私有页面，这不包括与系统或者其他应用共享的页面。

内容计数包括：

* Java：从 Java 或者 Kotlin 代码中对象分配的内存。

* Native：从 C 或者 C++ 代码中对象分配的内存。

  即使在你的应用中不包括 C++ 代码，你也可能看到一些 native 内存被使用，这是因为 Android 本身会使用 native 内存去处理各种任务，比如当处理图片和其他图像时。

* Graphics：用于图形缓冲区队列的内存在屏幕上显示像素，包括 GL surfaces，GL textures 等等。（注意这里是和 CPU 共享的内存，并不是 GPU 独有的内存。）

* Stack：应用中 native 和 Java 堆栈使用的内存。这一般和应用中使用的线程数有关。

* Code：应用中代码和资源文件使用的内存，比如 dex 字节码，优化或者编译 dex 的代码，.so 依赖，和字体。

* Other：应用中系统不知道如何分配的内存。

* Allocated：应用中分配的  Java 或者 Kotlin 对象数量。这不包括分配的 C 或者 C++ 对象。

  当运行在 Android 7.1 或者 Android 7.1 以下的设备时，这里的分配数量只从 Memory Profiler 连接到你的应用时开始计数。所以一些在分析开始之前分配的对象数量不会被包括在这里。不过，Android 8.0 已经包括设备上的分析工具，来跟踪所有分配。所以在 Android 8.0 或 Android 8.0 以上的设备上，这里的数字总是表示未完成 Java 对象数量总和。

因为新的 Memory Profiler 还了其他类别的内存，所以和旧的 Android Monitor 工具相比，看上去使用的内存会更高一点。不过你只是关注 Java heap，那么 Java 类别显示的数值和 Android Monitor 显示内存数值相似。

> 注意：当前，Memory Profiler 会在你的应用的分析中显示一些实际属于分析工具的 native 内存使用，10 MB 内存大概会添加 ~100k 对象。在之后的版本中，会解决了这些数值。



## 展示内存分配

Memory Profiler 会展示以下关于对象分配的情况：

* 什么类型的对象被分配，以及它们使用了多少空间。
* 每个线程中，每个分配的堆栈跟踪。
* 当对象被重新分配时（只有运行在 Android 8.0 或者 之上）。

当运行在 Android 8.0 或者以上的设备时，可以通过直接选择时间轴来查看任何时间的对象分配情况。

当运行在 Android 7.0 或者以下的设备时，可以通过点击录制内存分配按钮，来录制该段时间的内存分配情况。

> 当运行在 Android 7.0 或者以下的设备时，最多只能记录 65535 条对象分配。

![view memory allocations](https://developer.android.com/studio/images/profile/memory-profiler-allocations-detail_2x.png)



## 捕获 heap dump

当你捕获一个 heap dump 文件时，你可以查看以下：

* 你的应用分配了什么类型的对象，以及每种类型分配的大小。
* 每个对象使用了多少内存。
* 每个对象引用在你的代码中哪个地方持有。
* 对象分配的调用栈。（调用栈只有在使用 Android 7.1 或者以下，使用录制内存分配情况捕获的 heap dump 时可见）

你可以通过点击 Dump Java heap 按钮来捕获 heap dump，也可以通过代码中调用 `dumpHprofData()` 来获取更 精确的 heap dump。

![heap dump](https://developer.android.com/studio/images/profile/memory-profiler-dump_2x.png)



在类列表中，你可以获取以下信息：

* Alloc Count：堆中分配的数量。
* Native Size：该类型对象总共使用的 native 内存（以字节为单位）。这个只在 Android 7.0 以及以上可见。你将看到在 Java 中有些对象的内存在这里被分配，因为 Android 在一些 framework 类会使用 native 内存，比如 `Bitmap`。
* Shallow Size：该类型对象总共使用的 Java 内存（以字节为单位）。
* Retained Size：由于此类的所有实例而保留的内存总大小（以字节为单位）。

在类列表的顶部，你可以通过选择下拉框来选择展示不同的 heap dumps：

* Default heap：当系统没有指定堆时。
* App heap：应用分配内存的私有堆。
* Image heap：系统启动映像，包含在启动时预加载的类。这里的分配保证不会移动或消失。
* Zygote heap：应用进程从 Android 系统中 fork 时的 copy-on-write 的堆。

类列表默认是按类名分组，你可以选择不同的分组：

* Arrange by class：按类名分组。
* Arrange by package：按包名分组。
* Arrange by callstack：按调用栈分组。只有在录制分配时捕获 heap dump 时才有效果。

点击某个类名，打开该类的实例查看，从每个对象实例中你可以获取：

* Depth：该实例距离 GC 根节点的最短路径数。
* Native Size：该实例在 native 内存大小。只有在 Android 7.0 及以上可见。
* Shallow Size：该实例在 Java 内存中的大小。
* Retained Size：该实例占据的内存大小。（因为该实例而存在的内存）

> 默认情况下，heap dump 不可以展示每个分配对象的调用栈跟踪。要想获取调用栈跟踪，你必须在点击 Dump Java heap 之前，先开始录制内存分配，然后你就可以选择某个实例，同时在 Call Stack 选项中查看。无论如何，有一些对象是在录制之前分配的，所以这些对象的调用栈是不可用的。包含调用栈的实例会有个 icon。（在 Android 8.0 无法查看调用栈）

![call stack](https://developer.android.com/studio/images/profile/memory-profiler-dump-stacktrace_2x.png)

通过以下步骤来检测堆：

1. 浏览列表以查找具有异常大堆计数并可能泄漏的对象。对于已知对象，点击类名，查看每个实例。
2. 在 Instance View 面板中，点击某个实例，可以通过 References 选项查看对于该实例的每个引用。或者，点击实例旁边的箭头来查看实例的所有字段，并且点击某个字段名来查看字段的所有引用。如果你想要查看某个字段的具体实例，右击选择 Go to Instance。
3. 在 References 选项中，如果你想检查可能内存泄露的某个引用，右击选择 Go to Instance。这将从 heap dump 中选择相应的实例，显示这个实例的数据。

可能引起内存泄露的情况：

* 生命周期长的对象引用了 Activity，Context，View，Drawable 和其他可能持有 Activity 或者 Context 的引用。
* 非静态内部类。比如 `Runnable` 持有了 Activity 实例。
* 缓存超过必须时间持有对象。



### 保存 heap dump

可以通过 Export capture to file 来导出 .hprof 后缀文件。

还可以使用 android_sdk/platform-tools/ 下的 prof-conv 来转化成不同格式。



## 分析内存的技巧

可以通过设备旋转，来使得 Activity 重建，更快的复现可能的内存泄露。

> 也可以通过 `monkeyrunner` 测试框架。