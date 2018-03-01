# Java 并发基础



## 并发编程的三个概念

* 原子性

  即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

* 可见性

  当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

* 有序性

  即程序执行的顺序按照代码的先后顺序执行。

  > 指令重排序：一般来说，处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。
  >
  > 指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性。

要想并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。



## Java 内存模型与并发编程

为了获得较好的执行性能，Java 内存模型并没有限制执行引擎使用处理器的寄存器或者高速缓存来提升指令执行速度，也没有限制编译器对指令进行重排序。也就是说，在 java 内存模型中，也会存在缓存一致性问题和指令重排序的问题。

Java 语言 本身对 原子性、可见性以及有序性提供了哪些保证呢？

* 原子性

  在 Java 中，对基本数据类型和引用类型变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行，要么不执行。不过这里需要注意的是，Java 并不保证对 `long` 和 `double` 等 64 位数据类型的操作是原子性操作，存在有的虚拟机实现的操作是原子性，有的不是。

  > For the purposes of the Java programming language memory model, a single write to a non-volatile `long` or `double` value is treated as two separate writes: one to each 32-bit half. This can result in a situation where a thread sees the first 32 bits of a 64-bit value from one write, and the second 32 bits from another write.
  >
  > 在 Java 编程语言内存模型中，对非 valatile 的 long 或 double 的值的写入被视为两个独立的写入，一次为 32位。
  >
  > Writes and reads of volatile `long` and `double` values are always atomic.
  >
  > 写入或者读取 volatile long 或 double 值总是原子性操作。
  >
  > Writes to and reads of references are always atomic, regardless of whether they are implemented as 32-bit or 64-bit values.
  >
  > 写入或者读取一个引用类型总是原子性操作，不管它引用的值是 32 位还是 64 位。
  >
  > Some implementations may find it convenient to divide a single write action on a 64-bit `long` or `double` value into two write actions on adjacent 32-bit values. For efficiency's sake, this behavior is implementation-specific; an implementation of the Java Virtual Machine is free to perform writes to `long` and `double` values atomically or in two parts.
  >
  > 为了提高效率，Java 虚拟机的实现可以自由选择对 long 和double 值的写入是原子性操作或者是分为两个部分。
  >
  > Implementations of the Java Virtual Machine are encouraged to avoid splitting 64-bit values where possible. Programmers are encouraged to declare shared 64-bit values as `volatile` or synchronize their programs correctly to avoid possible complications.
  >
  > Java 虚拟机的实现在可能的情况下，应该避免对 64 位数据的切割操作。开发人员应该尽可能使用 volatile 或者使用同步操作。

  在 JSR-133 之前的规范中，64 位数据读也是分成了两个 32 位的读，但是从 JSR-133 规范开始，即 JDK5 开始，读操作也都具有原子性。

* 可见性

  对于可见性，Java 提供了 volatile 关键字来保证可见性。当一个共享变量被 volatile 修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。

  通过 synchronized 和 Lock 也能够保证可见性，synchronized 和 Lock 能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性。

* 有序性

  在 Java 内存模型中，允许编译器和处理器对指令进行重排序，但是重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

  在 Java 中，可以通过 volatile 关键字来保证一定的有序性，也可以通过 synchronized 和 Lock 来保证。synchronized 和 Lock 保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。

  Java 内存模型具备一些先天的“有序性”，即不需要通过任何手段就能够得到保证的有序性，这个通常也称为 happens-before 原则。如果两个操作的执行次序无法从 happens-before 原则推导出来，那么它们就不能保证它们的有序性，虚拟机可以随意地对它们进行重排序。

  * 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作。（一段程序代码的执行在单个线程中看起来是有序的）
  * 锁定规则：一个 unLock 操作先行发生于后面对同一个锁 lock 操作。
  * volatile 变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作。
  * 传递规则：如果操作 A 先行发生于操作 B，而操作B又先行发生于操作 C，则可以得出操作 A 先行发生于操作 C。
  * 线程启动规则：Thread 对象的 `start()` 方法先行发生于此线程的每个一个动作。
  * 线程中断规则：对线程` interrupt()` 方法的调用先行发生于被中断线程的代码检测到中断事件的发生
  * 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过 `Thread.join()` 方法结束、`Thread.isAlive()` 的返回值手段检测到线程已经终止执行。
  * 对象终结规则：一个对象的初始化完成先行发生于它的 `finalize()` 方法的开始。