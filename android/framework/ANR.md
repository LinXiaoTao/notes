> 基于 Android-27

## 检测

有四种情况会造成 ANR(Application Not reponding)：

* ServiceTimeout

  ``` java
  // 服务完成执行包括：onCreate，onStartCommand，onBind
  // 前台服务完成执行超时时间
  static final int SERVICE_TIMEOUT = 20*1000;
  // 后台服务完成执行超时时间
  static final int SERVICE_BACKGROUND_TIMEOUT = SERVICE_TIMEOUT * 10;

  // startForegroudService 执行超时时间
  static final int SERVICE_START_FOREGROUND_TIMEOUT = 5*1000;
  ```

  检测超时步骤：

  1. `ActiveServices.scheduleServiceTimeoutlocked()` 延迟发送 `SERVICE_TIMEOUT_MSG`

     ``` java
     // 调用时机为 调用 Service 的 onCreate，onStartCommand，onBind 之前
     void scheduleServiceTimeoutLocked(ProcessRecord proc) {                         
         if (proc.executingServices.size() == 0 || proc.thread == null) {            
             return;                                                                 
         }                                                                           
         Message msg = mAm.mHandler.obtainMessage(                                   
                 ActivityManagerService.SERVICE_TIMEOUT_MSG);                        
         msg.obj = proc;                                                             
         mAm.mHandler.sendMessageDelayed(msg,                                        
                 proc.execServicesFg ? SERVICE_TIMEOUT : SERVICE_BACKGROUND_TIMEOUT);
     }                                                                                
     ```

  2. `ActiveServices.serviceDoneExecutingLocked()` 取消 `SERVICE_TIMEOUT_MSG`

     ``` java
     // 调用时机为 执行完 Service 的 onCeate 或 onStartCommand
     mAm.mHandler.removeMessages(ActivityManagerService.SERVICE_TIMEOUT_MSG, r.app);
     ```

* BroadcastQueueTimeout

  有序广播才会检测超时

  ` sendOrderBroadcast`

  ``` java
  // 前台广播
  static final int BROADCAST_FG_TIMEOUT = 10*1000;
  // 后台广播
  static final int BROADCAST_BG_TIMEOUT = 60*1000;
  ```

  检测超时步骤：

  如果当前时间已经超过了 2 * receiver 个数 * timeout，则会触发 ANR。 

  1. `BroadcastQueue.setBroadcastTimeoutLocked` 延迟发送 `BROADCAST_TIMEOUT_MSG`

     ``` java
     final void setBroadcastTimeoutLocked(long timeoutTime) {                       
         if (! mPendingBroadcastTimeoutMessage) {                                   
             Message msg = mHandler.obtainMessage(BROADCAST_TIMEOUT_MSG, this);     
             mHandler.sendMessageAtTime(msg, timeoutTime);                          
             mPendingBroadcastTimeoutMessage = true;                                
         }                                                                          
     }                                                                              
     ```

  2. `BroadcastQueue.cancelBroadcastTimeoutLocked` 取消 `BROADCAST_TIMEOUT_MSG`

     ``` java
     final void cancelBroadcastTimeoutLocked() {                         
         if (mPendingBroadcastTimeoutMessage) {                          
             mHandler.removeMessages(BROADCAST_TIMEOUT_MSG, this);       
             mPendingBroadcastTimeoutMessage = false;                    
         }                                                               
     }                                                                   
     ```

* ContentProviderTimeout

  ``` java
  // 等待 attach 进行去发布它的 content providers 的时间，在发布之前我们先挂起
  static final int CONTENT_PROVIDER_PUBLISH_TIMEOUT = 10*1000;
  ```

  检测超时步骤：

  1. `ActivityManagerService.attachApplicationLocked` 中延迟发送 `CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG`

     ``` java
     if (providers != null && checkAppInLaunchingProvidersLocked(app)) {              
         Message msg = mHandler.obtainMessage(CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG);  
         msg.obj = app;                                                               
         mHandler.sendMessageDelayed(msg, CONTENT_PROVIDER_PUBLISH_TIMEOUT);          
     }                                                                                
     ```

  2. `ActivityManagerService.publishContentProviders` 中取消 `CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG`

     ``` java
     if (wasInLaunchingProviders) {                                       
         mHandler.removeMessages(CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG, r);
     }                                                                   
     ```

* InputDispatchingTimeout

  待完善。。。

## 处理

* 对于 Service，Broadcast，Input 发生了 ANR，最终会调用 `AppErrors.appNotResponding`
  1. 收集 CPU 信息
  2. 将 Log 写入 EventLog
  3. Log 输出到  main log
  4. 收集 stack traces 到 traces 文件，并保存到 DropBox，文件命名规则为：**anr_yyyy-MM-dd-HH-mm-ss-SSS**
  5. 如果是后台 ANR，直接杀死进程。否则，显示无响应对话框。


* 而 ContentProvider 在其进程启动时 publish 过程中可能会出现 ANR，则会杀死进程以及清理相应信息，而不会弹出 ANR 对话框。具体处理为：`ActivityManagerService.processContentProviderPublishTimedOutLocked()`

  ​