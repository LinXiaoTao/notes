[参考](https://github.com/CyC2018/InterviewNotes/blob/master/notes/JVM.md)

### JVM 内存模型

![内存模型](https://github.com/CyC2018/InterviewNotes/raw/master/pics/dc695f48-4189-4fc7-b950-ed25f6c80f82.jpg)

> 白色区域是线程私有，蓝色区域为线程共享

* Program Counter Register

  记录正在执行的 VM 字节码指令的地址，如果正在执行的是 native 方法则为空

* VM Stack

  每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每个方法从调用直到执行完成的过程，就对应一个栈帧在 VM Stack 中入栈和出栈的过程。

  该区域可能抛出以下异常：

  * 当线程请求的栈深度超过最大值，会抛出 `StackOverflowError`
  * 当栈进行动态扩展时没有申请到足够内存，会抛出 `OutOfMemoryError`

* Native Method Stack

  与 VM Stack 类似，区别在于 Native Method Stack 是为 native 方法使用

* Heap

  所有的对象实例都在这里分配内存

  这块区域是垃圾收集器管理的主要区域，即 GC Heap。现在的垃圾收集器基本采用分代收集算法，Heap 可以分为：新生代和老年代

  > 新生代还可以分为：Eden、From Survivor、To Survivor 等

  Heap 不需要连续地址的内存空间，可以通过 -Xmx 和 -Xms 来控制动态扩展内存大小，如果动态扩展失败则会抛出 `OutOfMemoryError`

* Method Area

  用于存储已被 VM 加载的类信息、常量、静态常量、即时编译的代码等数据

  和 Heap 一样不需要连续地址的内存空间，并且可以动态扩展，扩展失败一样会抛出 `OutOfMemoryError`

  对这块区域进行垃圾回收主要是对 Constant Pool 的回收和对类的卸载，但是一般比较难实现，HotSpot VM 把这块区域当成了永久代来进行垃圾回收

  * Runtime Constant Pool

    类加载后，Class 文件中 Constant Pool 就会被放到这个区域，Constant Pool 会存储编译时生成的各种字面量和符号引用

    在运行时可以通过 `String.intern()` 将新的常量放入该区域

* Direct Memory

  在 JDK 1.4 中新加入了 NIO 类，引入了一种基于 Channel 和 Buffer 的 IO，它可以使用 Native 函数库直接分配 Heap 外内存，然后通过一个存储在 Heap 中的 `DirectByteBuffer` 对象作为这块内存的引用进行操作，这样可以在一些场景中显著提高性能，因为避免在 Java Heap 和 Native Heap 中来回复制数据

### 垃圾收集

Program Counter Register、VM Stack、Native Method Stack 这三个区域属于线程私有，只存在线程的生命周期内，因此垃圾回收主要是针对 Heap 和 Method Area 进行。

#### 判断是否可回收

* 引用计数

  给对象添加一个引用计数器，引用数为 0 的对象可被回收

  如果两个对象相互引用，会导致无法回收

* 可达性

  通过 GC Roots 作为起始点进行搜索，能够到达的对象都是可用的，不可达的对象可被回收

  GC Roots 一般包含以下内容：

  * VM Stack 中引用的对象
  * Method Area 中类静态属性引用的对象
  * Method Area 中常量引用的对象
  * Native Method Stack 中引用的对象

#### 引用类型

Java 中存在四种引用类型，强度从高到低：

* 强引用

  默认引用类型，垃圾回收器永远不会回收被引用的对象

* 软引用（SoftReference）

  内存溢出之前进行回收

* 弱引用（WeakReference）

  只能生存到下一次垃圾收集发生之前，当垃圾收集器工作时，无论当前内存是否足够，都会被回收

* 虚引用（PhantomReference）

  完全不会对生存时间构成影响，也无法获取对象实例

  唯一目的就是能在对象被回收之前收到一个系统通知

#### Method Area 垃圾回收

在这个区域主要是对 Constant Pool 的回收和对类的卸载

Constant Pool 的回收和 Heap 中对象回收类似

类的卸载需要满足以下三个条件，并且满足了也不一定会被卸载：

1. 该类的所有实例都已经被回收
2. 该类的 ClassLoader 已经被回收
3. 该类对应的 Class 没有在任何地方被引用，也就无法在任何地方通过反射访问该类方法

#### finalize()

要真正回收一个对象，至少要经历两次标记过程：

如果对象在可达性分析后发现为不可达对象，那它会被**第一次标记**并且进行一次筛选，此对象是否有必要执行 `finaliza()`，当类没有覆盖 `finaliza()` 或者 `finaliza()` 已经被调用过，那么 JVM 认为没必要执行。

如果这个对象被判定为有必要执行 `finaliza()`，那么此对象将会被放置在 `F-Queue` 的队列中，并由 JVM 创建的、低优先级的 Finalizer 线程去执行它，如果一个对象在 `finaliza()` 中执行缓慢或者发生死循环等情况，将可能导致 F-Queue 中其他对象处于永久等待中，甚至导致整个回收系统奔溃

如果在 `finaliza()` 中重新变为可达对象，那么在**第二次标记**中将逃脱被回收的命运

**需要特别注意的是，并不提倡使用 `finaliza()`，因为它运行代价高昂，不确定性高，无法保证各个对象的调用顺序，而且只能被 JVM 调用一次**

#### 垃圾收集算法

* 标记-清除算法

  将需要回收的对象进行标记，然后清除

  缺点：

  1. 效率不高
  2. 产生大量的碎片

  ![标记-清除算法](https://github.com/CyC2018/InterviewNotes/raw/master/pics/a4248c4b-6c1d-4fb8-a557-86da92d3a294.jpg)

* 复制算法

  将内存划分为两块，每次只使用一块，当这一块使用完将还存活的对象复制到另一块上面，然后再清理使用的这一块内存

  缺点：

  1. 内存使用率不高
  2. 需要进行复制操作

  ![复制算法](https://github.com/CyC2018/InterviewNotes/raw/master/pics/902b83ab-8054-4bd2-898f-9a4a0fe52830.jpg)

* 标记-整理算法

  让所有存活对象都向一端移动，然后直接清理端边界以外的内存

  ![标记-整理算法](https://github.com/CyC2018/InterviewNotes/raw/master/pics/902b83ab-8054-4bd2-898f-9a4a0fe52830.jpg)

#### 分代收集算法

一般将 Heap 分为 Young Generation、Old Generation

* Young Generation 使用复制算法
* Old Generation 使用标记-清理算法或者标记-整理算法

将 Method Area 划分为 Permanent Generation

> Java 8 中 将 Permanent Generation 替换为 MetaSpace

所有新生成的对象首先放在 Young Generation，其中划分为三个区域 Eden、From Survivor、To Survivor

> Hotspot JVM 中 Eden 和 Survivor 的比例为 8:1

#### 内存分配与回收策略

* 对象优先在 Eden 分配

  大多数情况下，对象在 Young Generation 的 Eden 中进行分配，当 Eden 没有足够空间时，JVM 发起 Minor GC

* 大对象直接进入 Old Generation

  所谓大对象是指，需要大量连续内存空间的对象。JVM 提供了 -XX:PretenureSizeThreshold 参数，令大于这个值的对象直接分配在 Old Generation，避免在 Eden 和 Survivor 中发生大量的内存复制

* 长期存活的对象将进入 Old Generation

  JVM 给对象定义了一个 Age 计数，如果对象在 Eden 分配并经过一次 Minor GC 后仍然存储，并且能被 Survivor 容纳，将被移动到 Survivor 区域，Age 设置为 1，对象在 Survivor 中每经历一次 Minor GC，则 Age + 1，当增加到一定程度（默认 15），就会进入 Old Generation，这个值可以通过 -XX:MaxTenuringThreshold 设置

* 动态对象 Age 判定

  如果在 Survivor 区域中相同 Age 的所有对象大小总和大于 Survivor 空间的一半，大于等于该 Age 的对象可以直接进入 Old Generation

* 分配担保

  在发生 Minor GC 之前，JVM 会检查 Old Generation 最大可用的连续空间是否大于 Young Generation 中所有对象的总和，如果成立，则 Minor GC 是安全的，如果不成立，则 JVM 会检查 HandlePromotionFailure 是否允许分配担保失败，如果允许，那么会继续检查 Old Generation 最大可用的连续空间是否大于历次进入到 Old Generation 的对象的平均大小，如果大于，将继续 Minor GC，如果小于或者不允许分配担保失败，则改为 Full GC

  注意：如果出现分配担保失败，会在之后重新发起一次 Full GC

#### Minor GC

在 Young Generation 中发生的 GC，称为 Minor GC，触发条件为，当 Eden 区域满时

每次 Minor GC 都是用 Eden 和 From Survivor，当回收时，将这两块区域中还存活的对象都复制到 To Survivor，最后清理这两块区域。一次 Minor GC 结束后，Eden 和 From Survivor 区域都是空，而 To Survivor 存放着存活的对象，在下一次 Minor GC 时，两个 Survivor 交换标签，空的 From Survivor 标记为 To Survivor

#### Full GC

在 Young Generation、Old Generation、Permanent Generation 中发生的 GC，称为 Full GC

Full GC 的触发条件比较复杂：

* 调用 `System.gc()`

  调用这个函数建议 JVM 进行 Full GC，很多情况下会触发 Full GC，可以通过 -XX:DisableExplicitGC 来禁止 RMI 调用 `System.gc()`

* Old Generation 空间不足

  注意当 Full GC 后仍然空间不足，则会抛出 `OutOfMemoryError`

* 分配担保失败

  出现 HandlePromotionFailure，会触发 Full GC

* Java 8 之前的 Permanent Generation 空间不足

  Java 8 之前的 Method Area 属于 Permanent Generation，如果 Full GC 后仍然空间不足，则会抛出 `OutOfMemoryError`

* Concurrent Mode Failure

  执行 CMS GC 的过程中同时有对象放入 Old Generation，而此时 Old Generation 因为浮动垃圾过多导致暂时空间不足，便会出现 Concurrent Mode Failure 错误，并触发 Full GC

#### 垃圾收集器

HotSpot JVM 提供了七种垃圾收集器，连线表示可以配合使用

![垃圾收集器](https://github.com/CyC2018/InterviewNotes/raw/master/pics/c625baa0-dde6-449e-93df-c3a67f2f430f.jpg)

> 并行和并发的概念：
>
> * 并行（Parallel）：指多个垃圾收集线程并行工作，但用户线程仍然处于等待状态
> * 并发（Concurrent）：用户线程和垃圾收集线程同时执行（不一定时并行，可能是交替执行）
>
> 吞吐量指 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值

* Serial

  单线程的 Young Generation 收集器，使用复制算法，在进行垃圾收集时，必须 Stop-The-World，主要用于 Client 模式

  优点是简单高效，没有线程交互的开销

  ![Serial](https://github.com/CyC2018/InterviewNotes/raw/master/pics/22fda4ae-4dd5-489d-ab10-9ebfdad22ae0.jpg)

  > Old Generation 使用 Serial Old

* ParNew

  Serial 的多线程版本，是 Server 模式下 JVM 首选的 Young Generation 收集器，使用复制算法，有一个很重要的原因是只有它能与 CMS 收集器配合

  默认开始的线程数量与 CPU 数量相同，可以使用 -XX:ParallelGCThreads 来设置

  ![ParNew](https://github.com/CyC2018/InterviewNotes/raw/master/pics/81538cd5-1bcf-4e31-86e5-e198df1e013b.jpg)

  > Old Generation 使用 Serial Old

* Parallel Scavenge

  并行的 Young Generation 收集器，是一个吞吐量优先的收集器，使用复制算法，主要适合在后台运算而不需要太多交互的任务

  可以使用 -XX:MaxGCPauseMillis 来设置最大垃圾收集停顿时间，-XX:GCTimeRatio（0 < value < 100 的整数）设置吞吐量大小。还可以使用 -XX:UseAdaptiveSizePolicy 来开启 GC 自适应的调节策略，自动设置 -XX:SurvivorRatio（Eden 和 Survivor 比例）、-Xmn（Young Generation 空间）、-XX:PretenureSizeThreshold（进入 Old Generation 对象的 Age）等等。

* Serial Old

  Serial 的 Old Generation 版本，同样也是单线程，使用标记-整理算法

  主要意义是 Client 模式下使用，在 Server 模式下，它有两大用途：

  * JDK 1.5 以及之前的版本（Parallel Old 出现之前）中与 Parallel Scavenge 搭配使用
  * 在 Concurrent Mode Failure 出现时，作为 CMS 的后备预案

  ![Serial Old](https://github.com/CyC2018/InterviewNotes/raw/master/pics/08f32fd3-f736-4a67-81ca-295b2a7972f2.jpg)

  > Young Generation 使用 Serial

* Parallel Old

  Parallel Scavenge 的 Old Generation 版本，同样是多线程，使用标记-整理算法

  注意这个收集器在 JDK 1.6 才开始提供，在此之前使用 Serial Old 配合

  ![Parallel Old](https://pic.yupoo.com/crowhawk/9a6b1249/b1800d45.png)

  > Young Generation 使用 Parallel Scavenge

* CMS (Concurrent Mark Sweep)

  Old Generation 的并发收集器，使用标记-清除算法

  GC 流程：

  1. 初始标记（initial mark）：仅仅标记一下 GC Roots 能关联的对象，速度很快，需要 Stop-The-World
  2. 并发标记（concurrent mark）：进行 GC Roots Tracing，耗时最长，不需要 Stop-The-World
  3. 重新标记（remark）：修正并发标记过程中导致标记产生变动的那一部分对象的标记，需要 Stop-The-World
  4. 并发清除（concurrent sweep）：不需要 Stop-The-World

  缺点：

  1. 对 CPU 资源敏感。CMS 默认启动 (CPU + 3) / 4 的线程数，当 CPU 不足 4 个时，CMS 对用户程序的影响就可能变得很大

  2. 无法处理浮动垃圾（Floating Garbage），浮动垃圾无法保存，可能出现 Concurrent Mode Failure 而临时启用 Serial Old 来进行 Full GC

     > 在并发清理阶段，用户线程出现的新的垃圾称为浮动垃圾

  3. 标记-清除算法导致的空间碎片

  ![CMS](https://github.com/CyC2018/InterviewNotes/raw/master/pics/62e77997-6957-4b68-8d12-bfd609bb2c68.jpg)

* G1 (Garbage-First)

  面向服务端的收集器，用于替换 CMS

  具备以下特点：

  * 并发与并行
  * 分代收集
  * 空间整合，整体来看是基于标记-整理算法，从局部 Region 上看是基于复制算法
  * 可预测的停顿

  在 G1 之前的其他收集器进行收集的范围是整个 Young Generation 或者 Old Generation，而 G1 是将 Heap 划分为多个大小相等的独立 Region

  之所以能建立可预测的停顿时间模型，是因为它可以有计划地避免在整个 Java 堆中进行全区域的垃圾收集。它跟踪各个 Region 里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region（这也就是 Garbage-First 名称的来由）。这种使用 Region 划分内存空间以及有优先级的区域回收方式，保证了它在有限的时间内可以获取尽可能高的收集效率。

  Region 不可能是孤立的，一个对象分配在某个 Region 中，可以与整个 Java 堆任意的对象发生引用关系。在做可达性分析确定对象是否存活的时候，需要扫描整个 Java 堆才能保证准确性，这显然是对 GC 效率的极大伤害。为了避免全堆扫描的发生，每个 Region 都维护了一个与之对应的 Remembered Set。虚拟机发现程序在对 Reference 类型的数据进行写操作时，会产生一个 Write Barrier 暂时中断写操作，检查 Reference 引用的对象是否处于不同的 Region 之中，如果是，便通过 CardTable 把相关引用信息记录到被引用对象所属的 Region 的 Remembered Set 之中。当进行内存回收时，在 GC 根节点的枚举范围中加入 Remembered Set 即可保证不对全堆扫描也不会有遗漏。

  如果不计算维护 Remembered Set 的操作，GC 流程：

  1. 初始标记（initial mark）：仅仅只是标记 GC Roots 能直接关联到的对象，并且修改 TAMS (Next Top Mark Start) 的值，让下一阶段用户线程并发运行时能在正确的 Region 中创建对象，需要 Stop-The-World
  2. 并发标记（concurrent mark）：进行 GC Roots Tracing，耗时最长，不需要 Stop-The-World
  3. 最终标记（final mark）：修正并发标记过程中导致标记产生变动的那一部分对象的标记，并且将对象变化记录在线程的 Remembered Set Logs，最终将 Logs 合并到 Remembered Set 中，需要 Stop-The-World，但可以并行执行
  4. 筛选回收（live data counting and evacuation）：对各个 Region 中回收价值和成本进行排序，根据用户期望的 GC 停顿时间制定回收计划，可以并发执行，但因为只回收部分 Region，时间可控，所以使用 Stop-The-Wolrd

  ![G1](https://pic.yupoo.com/crowhawk/53b7a589/0bce1667.png)

  ​

### 类加载机制

类的生命周期包括以下 7 个阶段：

* 加载（Loading）
* 验证（Verification)
* 准备（Preparation）
* 解析（Resolution）
* 初始化（Initialization）
* 使用（Using）
* 卸载（Unloading）

> 解析过程在某些情况下，可以在初始化之后开始，这是为了支持 Java 动态绑定

![class](https://github.com/CyC2018/InterviewNotes/raw/master/pics/32b8374a-e822-4720-af0b-c0f485095ea2.jpg)

#### 类初始化时机

有且只有下面五种情况必须对类进行初始化：

1. 遇到 new、getstatic、putstatic、invokestatic 这四条字节码指令时，具体场景是：使用 new 关键字实例化对象时、读取或设置类的静态字段（使用 final 修饰、已在 Constant Pool 中的静态字段除外）、调用类的静态方法等等
2. 使用 `java.lang.reflect` 对类进行反射调用时候
3. 初始化类时候，其父类还没进行初始化，先触发父类的初始化
4. JVM 启动时，包含 `main()` 的类
5. 使用 jdk 1.7 的动态语言支持，如果 `java.lang.invoke.MethodHandle` 实例最后的解析结果为 REF_getStatic、REF_putStatic、REF_invokeStatic 的方法句柄时

以上场景称为主动引用，除此之前，其他被动引用方式不会触发初始化，比如：

* 通过子类引用父类的静态字段
* 通过数组定义来引用类，数组类是由 JVM 自动生成、直接继承于 Object 的子类
* 常量在编译阶段会存入 Constant Pool

#### 类加载过程

* 加载

  这一阶段主要完成以下三件事情：

  * 通过一个类的全限定名来获取定义此类的二进制字节流
  * 将这个字节流所代表的静态存储结构转化为方法区的运行时存储结构
  * 在内存中生成一个代表这个类的 Class 对象，作为方法区这个类的各种数据的访问入口

* 验证

  确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全

  主要有以下 4 个阶段：

  * 文件格式验证
  * 元数据验证（对字节码描述的信息进行语义分析）
  * 字节码验证（通过数据流和控制流分析，确保程序语义是合法、符合逻辑的，将对类的方法体进行校验分析）
  * 符号引用验证

* 准备

  类变量是被 static 修饰的变量，准备阶段为类变量分配内存并设置初始值，使用的是方法区的内存。

  实例变量不会在这阶段分配内存，它将会在对象实例化时随着对象一起分配在 Java 堆中。

  ``` java
  // 在准备阶段，将被初始化为 0
  public static int value = 123;
  // 如果是常量，则被初始化为 123
  public static final int value = 123;
  ```

* 解析

  将常量池的符号引用替换为直接引用的过程

* 初始化

  初始化阶段即虚拟机执行类构造器 <clinit>() 方法的过程

  在准备阶段，类变量已经赋过一次系统要求的初始值，而在初始化阶段，根据程序员通过程序制定的主观计划去初始化类变量和其它资源

   <clinit>() 具有以下特点：

  * 是由编译器自动收集类中所有类变量的赋值动作和静态语句块（static{} 块）中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序决定。特别注意的是，静态语句块只能访问到定义在它之前的类变量，定义在它之后的类变量只能赋值，不能访问。

    ``` java
    public class Test {
        
        static {
            i = 0;
            System.out.println(i); //error
        }
        
        static int i = 1;
        
    }
    ```

  * 与类的构造函数（或者说实例构造器 <init>()）不同，不需要显式的调用父类的构造器。虚拟机会自动保证在子类的 <clinit>() 方法运行之前，父类的 <clinit>() 方法已经执行结束。因此虚拟机中第一个执行 <clinit>() 方法的类肯定为 java.lang.Object

  * <clinit>() 方法对于类或接口不是必须的，如果一个类中不包含静态语句块，也没有对类变量的赋值操作，编译器可以不为该类生成 <clinit>() 方法

  * 接口中不可以使用静态语句块，但仍然有类变量初始化的赋值操作，因此接口与类一样都会生成 <clinit>() 方法。但接口与类不同的是，执行接口的 <clinit>() 方法不需要先执行父接口的 <clinit>() 方法。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的 <clinit>() 方法

  * 虚拟机会保证一个类的 <clinit>() 方法在多线程环境下被正确的加锁和同步，如果多个线程同时初始化一个类，只会有一个线程执行这个类的 <clinit>() 方法，其它线程都会阻塞等待，直到活动线程执行 <clinit>() 方法完毕。如果在一个类的 <clinit>() 方法中有耗时的操作，就可能造成多个进程阻塞，在实际过程中此种阻塞很隐蔽

  #### 类加载器

  对于任意一个类，都需要由加载它的类加载器和类本身一同确立在 JVM 中的唯一性。

  内置类加载可以分为以下三种：

  * 启动类加载器（Bootstrap）：用 C++ 实现，负责加载存放在 `<JAVA_HOME>/lib` 或者使用 `-Xbootclasspath` 参数指定的路径中的，并且能被 JVM 识别的（比如 rt.jar）的类库

    用户编写自定义类加载器，如果需要将请求委派给 Bootstrap ClassLoader，直接使用 null 代替即可

  * 扩展类加载器（Extension）：用 Java (`sun.misc.Launcher$ExtClassLoader`) 实现，负责加载 `<JAVA_HOME>/lib/ext` 或者被 `java.ext.dir` 系统变量所指定的路径的类库

  * 应用程序类加载器（Application）：用 Java (`com.misc.Launcher$AppClassLoader`) 实现，使用 `ClassLoader.getSystemClassLoader()` 返回，负责加载用户路径（ClassPath）上指定的类库

  #### 双亲委托模型

  类加载器之间的层次关系，称为类加载器的双亲委托模型。

  除了 Bootstrap ClassLoader 以外，类加载器在收到了类加载的请求时，总是先委托给父类加载器，依次委托，只有父类加载器无法加载，才会尝试执行类加载请求

  ![ClassLoader](https://github.com/CyC2018/InterviewNotes/raw/master/pics/2cdc3ce2-fa82-4c22-baaa-000c07d10473.jpg)

  ​

