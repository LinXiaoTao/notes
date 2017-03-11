**Dalvik虚拟机**，是[Google](https://zh.wikipedia.org/wiki/Google)等厂商合作开发的[Android](https://zh.wikipedia.org/wiki/Android)移动设备平台的核心组成部分之一。它可以支持已转换为.dex（即“Dalvik Executable”）格式的[Java](https://zh.wikipedia.org/wiki/Java)应用程序的运行。.dex格式是专为Dalvik设计的一种压缩格式，适合[内存](https://zh.wikipedia.org/wiki/%E5%86%85%E5%AD%98)和[处理器](https://zh.wikipedia.org/wiki/%E5%A4%84%E7%90%86%E5%99%A8)速度有限的系统。

**字节码**（英语：Bytecode）通常指的是已经经过[编译](https://zh.wikipedia.org/wiki/%E7%BC%96%E8%AF%91)，但与特定[机器码](https://zh.wikipedia.org/wiki/%E6%A9%9F%E5%99%A8%E7%A2%BC)无关，需要[直译器](https://zh.wikipedia.org/wiki/%E7%9B%B4%E8%AD%AF%E5%99%A8)转译后才能成为[机器码](https://zh.wikipedia.org/wiki/%E6%A9%9F%E5%99%A8%E7%A2%BC)的[中间代码](https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%96%93%E8%AA%9E%E8%A8%80)。字节码通常不像[源码](https://zh.wikipedia.org/wiki/%E6%BA%90%E7%A2%BC)一样可以让人阅读，而是[编码](https://zh.wikipedia.org/wiki/%E7%B7%A8%E7%A2%BC)后的数值常量、引用、指令等构成的序列。

字节码主要为了实现特定软件运行和软件环境、与硬件环境无关。字节码的实现方式是通过[编译器](https://zh.wikipedia.org/wiki/%E7%B7%A8%E8%AD%AF%E5%99%A8)和[虚拟机器](https://zh.wikipedia.org/wiki/%E8%99%9B%E6%93%AC%E6%A9%9F%E5%99%A8)。编译器将源码编译成字节码，特定平台上的虚拟机器将字节码转译为可以直接执行的指令(可以简单理解为**字节码是虚拟机识别的格式**)。字节码的典型应用为[Java bytecode](https://zh.wikipedia.org/wiki/Java_bytecode)。

创建应用程序代码时，apk文件包含.dex文件，其中包含二进制Dalvik字节码。 这是平台实际理解的格式。 但是，读取或修改二进制代码并不容易，因此有一些工具可以转换为可读的表示形式。 最常见的人类可读格式称为Smali。这显然是更容易阅读二进制代码。 但是平台不知道什么关于smali，它只是一个工具，使它更容易使用字节码。(可以理解为**smali是源码和dalvik字节码的中间格式**)

* 大多数[虚拟机](https://zh.wikipedia.org/wiki/%E8%99%9A%E6%8B%9F%E6%9C%BA)包括[JVM](https://zh.wikipedia.org/wiki/JVM)都是一种[堆栈机器](https://zh.wikipedia.org/wiki/%E5%A0%86%E7%96%8A%E6%A9%9F%E5%99%A8)，而Dalvik虚拟机则是[寄存器机](https://zh.wikipedia.org/wiki/%E5%AF%84%E5%AD%98%E5%99%A8%E6%9C%BA)。两种架构各有优劣，一般而言，基于堆栈的机器需要更多指令，而基于寄存器的机器指令更长。
* 从[Android 5.0](https://zh.wikipedia.org/wiki/Android_5.0)版起，[Android Runtime](https://zh.wikipedia.org/wiki/Android_Runtime)（ART）替换Dalvik成为系统内默认虚拟机。
* dx工具是一种用来转换Java class成为DEX格式的工具。多个类被包含在一个dex文件之中。各个类中重复的字符串和其他常数只在DEX中存放一次，以节省空间。Java字节码（bytecode）被转换成Dalvik虚拟机所使用的替代指令集。一个未压缩dex文件通常稍小于一个已经压缩的.jar档。
* Dalvik虚拟机有自己的[字节码](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E7%A0%81)，并非使用Java字节码。
* Dalvik基于寄存器，而JVM基于堆栈。
* Dalvik VM通过Zygote进行类别的预加载，Zygote会完成虚拟机的初始化，也是与JVM不同之处。
* Dalvik和ART都采用包含dalvik字节码的.dex文件。 它对应用程序开发人员是完全透明的，唯一的区别是当应用程序安装和运行时幕后发生的。