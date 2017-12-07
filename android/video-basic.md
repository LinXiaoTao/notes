### 编码器

* MediaCodec

  可用于访问低级别媒体编解码器。可以通过 `createInputSurface` 方法设置 Surface 类型以及 `configure` 方法设置输出 Surface 类型。

* MediaRecorder

  用于记录音频和视频。可以通过 `setVideoSource` 方法设置视频源，两种：一种是摄像头，一种是录制屏幕。

这两种遍解码器的区别在于：MediaCodec 可以处理详细的视频流信息，而 MediaRecorder 使用简单，但不能接触到视频流数据，处理不了原生的视频数据。

### 视频源

* Camera

  用于设置图像捕获设置、启动／停止预览、快照和用于视频编码的检索帧。这个类是相机服务的客户端，用于管理真实的相机硬件。

  **android.hardware.Camera 废弃，使用 android.hardware.camera2**

* MediaProjection，VirtualDisplay

  MediaProjection 让应用程序可以捕获屏幕内容 和／或 记录系统音频。

  VirtualDisplay 通过 `setSurface` 方法来设置预览的 Surface。

### 视频源格式和视频编码数据格式

摄像头采集的视频数据格式是 N21 和 YV12，

屏幕视频录制的数据格式是 YUV420P(I420) 和 YUV420SP(N12)

编码器 MediaCodec 处理的数据格式是 YUV420P 和 YUV420SP* **待验证**

### 视频预览 View

* SurfaceView

  主要和 SurfaceHolder ，Surface 相关联，Camera 提供了 `setPreviewDisplay` 方法来设置 SurfaceHolder，VirtualDisplay 提供了 `setSurface` 来设置 Surface，可以通过以上方法来进行视频的预览。

* TextureView

  可以用来显示数据流，主要和 SurfaceTexture 相关联。Camera 提供了 `setPreviewTexture` 方法来设置 SurfaceTexture，VirtualDisplay 则没有，所以只能通过 Camera 来预览视频。

* GLSurfaceView

  继承于 SurfaceView，在这基础上添加 OpenGL 技术，用来处理数据。

### Surface

**处理由屏幕合成器管理的原始缓冲区**

Surface 通常由图像缓冲区 (例如：`SurfaceTexture` 和 `MediaRecorder` ，或者 `Allocation`) 的消费者创建，并被交给某种生产者 (如 `OpenGL`，`MediaPlayer` 或 `CameraDevice`) 进行绘制。

> 注意：Surface 与其相关联的消费者类似弱引用。本身不能保持它的 parent consumer 不被回收。

