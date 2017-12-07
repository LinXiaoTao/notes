[API](https://developer.android.com/reference/android/hardware/Camera.html)

[guide](https://developer.android.com/guide/topics/media/camera.html)

[参考](https://juejin.im/entry/58ca5fb62f301e007e3853ba)



### Camera.CameraInfo

**facing ** 表示相机的面向，可能的值有：

* **CAMERA_FACING_BACK** 相机的面向与屏幕相反
* **CAMERA_FACING_FRONT** 相机的面向与屏幕相同

**orientation** 表示相机图像的方向

这个值是相机图像需要顺时针旋转的角度，才能在显示屏上以自然方向正确显示。它应该是 0，90，180 或 270。这个值跟摄像头的位置有关。例如，假设屏幕是具有自然高的屏幕（高大于宽），后置摄像头的相机传感器安装在横行，面对屏幕，如果后置摄像头传感器的顶端与自然方向的屏幕右边缘对齐，则值应该为 90。如果前置摄像头传感器的顶部与屏幕右侧对齐，则值应该为 270。

#### 相关概念

* 设备自然方向。

  手机和平板的自然方向不同，`android:screenOrientation` 默认值 `unspecified` 就是自然方向。

  `landscape` 表示横向方向，显示的宽度大于高度，这是平板的自然方向

  `portrait` 表示纵向方向，显示的高度大于宽度，这是手机的自然方向

* 屏幕角度

  `Display.getRotation()` 返回屏幕从自然方向的旋转角度。

  返回值可能是 `Surface.ROTATION_0` 表示没有旋转，`Surface.ROTATION_90`，`Surface.ROTATION_180`，`Surface.ROTATION_270`。

  例如，如果屏幕具有自然方向高的屏幕（纵向方向），并且用户将其从侧面旋转为横向，则这里返回的值可能是 `Surface.ROTATION_90` 或 `Surface.ROTATION_270`，这取决于转动的方向。

  屏幕旋转角度和设备物理旋转是相反方向。例如，如果设备逆时针旋转 90 度，则这里返回 `ROTATION_90`，可以理解为，屏幕需要顺时针旋转多少度才能恢复自然方向。如果设备顺时针旋转 90 度，则屏幕需要逆时针旋转 90 度，也就是顺时针旋转 270 度，则这里返回 `ROTATION_270`。

* 屏幕尺寸

  [Display](https://developer.android.com/reference/android/view/Display.html) 提供了有关逻辑显示的大小和密度的信息。

  以两种不同的方式进行描述：

  应用程序显示区域指定可能包含应用程序窗口的显示部分，不包括系统装饰。应用显示区域可能小于实际显示区域，因为系统减去诸如状态栏的装饰元素所需的空间。使用以下方法查询应用程序显示区域：`getSize(Point)`，`getRectSize(Rect)` 和 `getMetrics(DisplayMetrics)`。

  真正的显示区域指定包含系统装饰内容的显示部分。即使如此，如果窗口管理器正在使用（adb shell wm size）模拟较小的显示，则实际显示区域可能小于显示器的物理大小。使用以下方法查询实际显示区域：`getRealSize(Point)`，`getRealMetrics(DisplayMetrics)`。

  逻辑显示不一定表示诸如内置屏幕或外部显示器等特定物理显示设备。逻辑显示内容可以根据当前附加的设备和是否启用了镜像来呈现在一个或多个物理显示器上。

* 预览角度

  `Camera.setDisplayOrientation(int)` 将预览显示按顺时针旋转。

  这个 API 会影响预览的帧数据和快照的图像，但不会影响 `onPreviewFrame(byte[],Camear)` 方法中，JPEG 图片或录制视频中传递的字节数组的顺序。

  不会在预览过程中不允许调用这个方法，但从 API 14 起，允许在预览过程中调用这个方法。

  在 API 24 之前，预览显示的方向默认为 0。从 API 24 开始，默认方向将使得强制为横向模式的应用程序具有正确的预览方向，这可能是 0 或 180，不过以纵向方向或允许改变方向的应用程序则仍然需要调用这个方法，确保所有情况下正确的预览显示。

  ``` java
  //官方推荐的设置预览方向 
  public static void setCameraDisplayOrientation(Activity activity,
           int cameraId, android.hardware.Camera camera) {
       android.hardware.Camera.CameraInfo info =
               new android.hardware.Camera.CameraInfo();
       android.hardware.Camera.getCameraInfo(cameraId, info);
       int rotation = activity.getWindowManager().getDefaultDisplay()
               .getRotation();
       int degrees = 0;
       switch (rotation) {
           case Surface.ROTATION_0: degrees = 0; break;
           case Surface.ROTATION_90: degrees = 90; break;
           case Surface.ROTATION_180: degrees = 180; break;
           case Surface.ROTATION_270: degrees = 270; break;
       }

       int result;
       if (info.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
           result = (info.orientation + degrees) % 360;
           result = (360 - result) % 360;  // compensate the mirror
       } else {  // back-facing
           result = (info.orientation - degrees + 360) % 360;
       }
       camera.setDisplayOrientation(result);
   }
  ```

* 预览方向和相机拍摄方向不一致的原因

  相机传感器方向的 X 轴和纵向屏幕预览方向的 X 轴：

  ![屏幕坐标系](https://mc.qcloudimg.com/static/img/4227cea040746fa0d7164881ab0fda17/image.jpg)

  通过上图，可以得知相机传感器方向的 X 轴和纵向屏幕预览方向的 X 轴呈 90 度。

  由于手机屏幕可以 360 度旋转，为了保证用户无论怎么旋转手机都能看到，手机屏幕内容与人眼看到的内容方向一致，Android 会根据手机屏幕的方向对屏幕内容进行旋转处理（`Display.getRotation()` 获取）。所以当手机旋转为横屏时（注意和反向横屏做区分），Android 会将屏幕内容也逆时针旋转 90 度来处理，而纵向屏幕方向和相机传感器方向也刚好是，将纵向屏幕逆时针旋转 90 度，因此，当我们为横向屏幕时，手机屏幕预览方向和相机传感器方向是一致的。

* 摄像头拍摄图像的旋转方向处理

  通过调用 `Camera.setRotation(int)` 可以设置相对于摄像头方向的顺时针旋转角度，这会影响从 `Camera.PictureCallBack` JPEG 返回的图像，相机驱动可以在 EXIF 信息中设置方向，而不真正旋转图像，或者真正旋转图像和 EXIF 缩略图。如果 JPEG 图像旋转，则 EXIF 信息中的方向将丢失 或者设置为 1。

  Android 相机有个特殊设定，对于前置摄像头，在预览时采用镜面成像的效果，而拍摄出来的图像仍采用摄像头成像。

  摄像头的拍摄图像旋转方向处理：

  ``` java
  public void onOrientationChanged(int orientation) {
       if (orientation == ORIENTATION_UNKNOWN) return;
       android.hardware.Camera.CameraInfo info =
              new android.hardware.Camera.CameraInfo();
       android.hardware.Camera.getCameraInfo(cameraId, info);
       orientation = (orientation + 45) / 90 * 90;
       int rotation = 0;
       if (info.facing == CameraInfo.CAMERA_FACING_FRONT) {
           rotation = (info.orientation - orientation + 360) % 360;
       } else {  // back-facing camera
           rotation = (info.orientation + orientation) % 360;
       }
       mParameters.setRotation(rotation);
   }
  ```

  ​

  ​

  ​
