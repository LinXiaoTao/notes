[API](https://developer.android.com/reference/android/hardware/Camera.html)

[guide](https://developer.android.com/guide/topics/media/camera.html)



### Camera.CameraInfo

**facing ** 表示相机的面向，可能的值有：

* **CAMERA_FACING_BACK** 相机的面向与屏幕相反
* **CAMERA_FACING_FRONT** 相机的面向与屏幕相同

**orientation** 表示相机图像的方向

这个值是相机图像需要顺时针旋转的角度，才能在显示屏上以自然方向正确显示。它应该是 0，90，180 或 270。

例如，如果屏幕方向是 portrait，那么后置摄像头这个值是 90，前置摄像头是 270。

#### 相关概念

* 设备自然方向。

  手机和平板的自然方向不同，`android:screenOrientation` 默认值 `unspecified` 就是自然方向。

  `landscape` 表示横向方向，显示的宽度大于高度，这是平板的自然方向

  `portrait` 表示纵向方向，显示的高度大于宽度，这是手机的自然方向

* 屏幕角度

  返回屏幕从自然方向的顺时针旋转的角度。返回值可能是 `Surface.ROTATION_0` 表示没有旋转，`Surface.ROTATION_90`，`Surface.ROTATION_180`，`Surface.ROTATION_270`。

  例如，如果屏幕具有自然方向高的屏幕（纵向方向），并且用户将其从侧面旋转为横向，则这里返回的值可能是 `Surface.ROTATION_90` 或 `Surface.ROTATION_270`，这取决于转动的方向。

  屏幕旋转角度和设备物理旋转是相反方向。例如，如果设备逆时针旋转 90 度，则这里返回 `ROTATION_90`，可以理解为，屏幕需要顺时针旋转多少度才能恢复自然方向。如果设备顺时针旋转 90 度，则屏幕需要逆时针旋转 90 度，也就是顺时针旋转 270 度，则这里返回 `ROTATION_270`。