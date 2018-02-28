参考 [gityuan](http://gityuan.com/2016/02/01/android-booting/)

### 概述

Android 系统底层基于 Linux Kernel，Kernel 启动过程会创建 init 进程，该进程是所有用户空间的鼻祖，init 进程会启动 ServiceManager，Zygoto 进程。Zygoto 进程会创建 system_server 进程以及各种 app 进程。

![概述](http://gityuan.com/images/process/android-booting.jpg)

### init

init 进程是 Linux 系统中用户空间的第一个进程（pid = 1），Kerner 启动后会调用 `/system/core/init/Init.cpp` 的 `main()` 方法。

init 进程的主要功能点：

1. 分析和运行所有的 init.rc 文件
2. 生成设备驱动节点（通过 rc 文件创建）
3. 处理子进程的终止（signal 方式）
4. 提供属性服务 property service

当 init 子进程退出时，会产生 SIGCHLD 信号，并发送给 init 进程，通过 socket 传递数据，调用到 `wait_for_one_process()`，根据是否是 onehot，来决定是否重启子进程，由于缺省模式 oneshot = false，因此子进程一旦被杀死便会由 init 进程重启。

![SIGCHLD](http://gityuan.com/images/boot/init/init_oneshot.jpg)

### zygote

init.rc 中根据 `ro.zygote` 会导入 32 位或者 64 位的 zygote.rc 文件

```
import /init.environ.rc
import /init.usb.rc
import /init.${ro.hardware}.rc
import /vendor/etc/init/hw/init.${ro.hardware}.rc
import /init.usb.configfs.rc
import /init.${ro.zygote}.rc
```

这是一份 init.zygote32.rc 文件

```
# 启动 Zygote 进程，启动成功后，开启 system-server 进程
service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
	# 伴随着main class的启动而启动
    class main
    priority -20
    user root
    group root readproc
    # 创建 socket
    socket zygote stream 660 root system
    onrestart write /sys/android_power/request_state wake
    onrestart write /sys/power/state on
    # zygote 重启时，重启以下
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    onrestart restart wificond
    writepid /dev/cpuset/foreground/tasks

```

当 Zygote 进程启动后，便会执行到 `/frameworks/base/cmds/app_process/app_main.cpp` 文件的 `main` 方法，整个调用流程：

```
app_main.main
	AndroidRuntime.start
		//启动虚拟机
		AndroidRuntime.startVm
		// JNI 方法注册
		AndroidRuntime.startReg
		// Java 世界
		ZygoteInit.main
			// 注册 socket
			ZygoteServer.registerServerSocket
			// 预加载类和资源
			ZygoteInit.preload
			// 启动 system_server 进程
			ZygoteInit.startSystemServer
			// 进入循环模式
			ZygoteServer.runSelectLoop	
```

Zygote 进程创建 Java 虚拟机，并注册 JNI 方法。在创建完 system_server 进程后，调用 runSelectLoop() 等待客户端的链接（socket）。

### system_server

Zygote 通过 fork 创建 system_server 进程

``` java
// ZygoteInit.java

private static void handleSystemServerProcess(ZygoteConnection.Arguments parsedArgs) {
    
    //...
    // 设置当前进程名为 system_server
    if (parsedArgs.niceName != null) {
            Process.setArgV0(parsedArgs.niceName);
     }
    
    //...
            ClassLoader cl = null;
            if (systemServerClasspath != null) {
                // 设置 SystemClassLoader
                cl = createPathClassLoader(systemServerClasspath, parsedArgs.targetSdkVersion);

                Thread.currentThread().setContextClassLoader(cl);
            }
    
    // 将剩余的参数传递给 SystemServer
    return ZygoteInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs, cl);
}

static ClassLoader createPathClassLoader(String classPath, int targetSdkVersion) {
        String libraryPath = System.getProperty("java.library.path");
		// ClassLoader.getSystemClassLoader 会创建 PathClassLoader 作为 SystemClassLoader 而不是 Java 默认的 URLClassLoader
        return ClassLoaderFactory.createClassLoader(classPath, libraryPath, libraryPath,
                ClassLoader.getSystemClassLoader(), targetSdkVersion, true /* isNamespaceShared */,
                null /* classLoaderName */);
    }
```

``` java
//ZygoteInit.java
public static final Runnable zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader) { 
                                                                                                     
    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ZygoteInit");
    // log 重定向
    RuntimeInit.redirectLogStreams();                                                                   
    // 通用初始化                                                                                                    
    RuntimeInit.commonInit(); 
    // zygote 初始化
    ZygoteInit.nativeZygoteInit();                                                                      
    return RuntimeInit.applicationInit(targetSdkVersion, argv, classLoader);                            
}                                                                                                       
```

`ZygoteInit.nativeZygoteInit()` 最终会进入 `app_main.app` 中的 `onZygoteInit()`

``` cpp
virtual void onZygoteInit() {
    sp<processState> proc = ProcessState::self();
    // 启动新的 Binder 线程
    proc->startThreadPool();
}
```

`RuntimeInit.applicationInit()` 会通过 `findStaticMain()` 返回一个调用 `SystemServer.main()` 的 `Runnable`（通过反射）。

``` java
//SystemServer.java
public static void main(String[] args) {
    new SystemServer().run();
}

private void run() {
    //...
    // 准备 main Looper
    Looper.prepareMainLooper();
    // Initialize native services.             
	System.loadLibrary("android_servers");
    // 检测上一次关机是否失败                               
	performPendingShutdown();                                  
    // 初始化 system context
    createSystemContext();
    // Create the system service manager.                                      
	mSystemServiceManager = new SystemServiceManager(mSystemContext);          
	mSystemServiceManager.setRuntimeRestarted(mRuntimeRestart);                
	LocalServices.addService(SystemServiceManager.class, mSystemServiceManager)
	// Prepare the thread pool for init tasks that can be parallelized         
	SystemServerInitThreadPool.get();
    //...
    // 开启引导服务，包括 ActivtyManagerService
    startBootstrapServices();
    // 开启核心服务
	startCoreServices();     
    // 开始其他服务
	startOtherServices(); 
    // ...
    // main looper 开始循环
    Looper.loop();
}
```

在 `ActivityManagerService.systemReady()` 中调用 `startHomeActivityLocked()` 启动 Home

### app

对于普通的 app 进程，跟 system_server 进程的启动有点类似，不同的是 app 进程是向 system_server 进程发送消息，由 system_server 向 zygote 发出创建进程的请求。

app 进程创建成功后，会进入 `ActivityThread.main()`

``` java
//ActivityThread.java
public static void main(String[] args) {                                              
    //...                           
    // 创建 main looper                                                                                  
    Looper.prepareMainLooper();                                                       
                                                                                      
    ActivityThread thread = new ActivityThread();                                     
    thread.attach(false);                                                             
                                                                                      
    if (sMainThreadHandler == null) {                                                 
        sMainThreadHandler = thread.getHandler();                                     
    }                                                                                 
                                                                                      
    // main looper 开始循环                                
    Looper.loop();                                                                    
                                                                                      
    throw new RuntimeException("Main thread loop unexpectedly exited");               
}

private void attach(boolean system) {                                                                              
    	//...
    	// IActivityManager 通过 Binder 调用 ActivityManagerService
        final IActivityManager mgr = ActivityManager.getService();                                                 
        try {
            // 调用 Application.onCreate
            mgr.attachApplication(mAppThread);                                                                     
        } catch (RemoteException ex) {                                                                             
            throw ex.rethrowFromSystemServer();                                                                    
        }                                                                                                          
        //...                                                                                            
    } else {                                                                                                       
       //...                                                               
    }                                                                                                              
                                                                                                                   
    //...                                                   
}                                                                                                                  
```









