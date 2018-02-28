**参考 [gityuan](http://gityuan.com/)**

### IPC 和 Binder

每个 Android 进程只能运行在自己进程所拥有的虚拟地址空间，而虚拟地址空间又分为用户空间和内核空间。用户空间之间彼此不能共享，而内核空间却是可以共享的。Binder 机制中进行 IPC 就是利用进程间可共享的内核空间来完成底层通信工作的。Client 端和 Server 端进程往往采用 ioctl 等方法跟内核空间的驱动进行交互。

![IPC](http://gityuan.com/images/binder/prepare/binder_interprocess_communication.png)

### Binder 原理

Binder 机制采用 C/S 架构，组件包括 Client、Server、ServiceManager 以及 Binder 驱动，其中 ServiceManager 用于管理系统中的各种服务，架构图如下所示：

![Binder 机制](http://gityuan.com/images/binder/prepare/IPC-Binder.jpg)

> 需要注意是的，这里说的 ServiceManager 指 Native 层的 ServiceManager(C++)，并非指 Java Framework 层的 ServiceManager(Java)。

图中 Client、Server、ServiceManager 之间的交互都是通过 Binder 驱动进行间接交互。其中 Binder 驱动位于内核空间，Client、Server、ServiceManager 位于用户空间。

Binder 驱动和 ServiceManager 可以看作是 Android 平台的基础架构，而 Client 和 Server 是 Android 应用层。

BpBinder（客户端）和 BBinder（服务端）是 Android 中 Binder 通信相关的类，关系图如下：

![Android Binder](http://gityuan.com/images/binder/prepare/Ibinder_classes.jpg)

 