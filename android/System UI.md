### FLAG
* `SYSTEM_UI_FLAG_IMMERSIVE`:设置沉浸式，需要配合`SYSTEM_UI_FLAG_FULLSCREEN`和`SYSTEM_UI_FLAG_HIDE_NAVIGATION`的使用来隐藏状态栏和导航栏
* `SYSTEM_UI_FLAG_IMMERSIVE_STICKY`:设置粘性沉浸式，和`SYSTEM_UI_FLAG_IMMERSIVE`的区别在于，非粘性沉浸向下滑动状态栏和向上滑动导航栏会清除掉`SYSTEM_UI_FLAG_FULLSCREEN`和`SYSTEM_UI_FLAG_HIDE_NAVIGATION`标记，重现状态栏和导航栏，而粘性沉浸会暂时出现状态栏和导航栏，又重新恢复。
* `SYSTEM_UI_FLAG_LAYOUT_STABLE`:在使用`SYSTEM_UI_FLAG_FULLSCREEN`和`SYSTEM_UI_FLAG_HIDE_NAVIGATION`隐藏系统UI后，重现系统UI时候，应用窗口会重新测量大小，而引起屏幕跳跃，这个flag可以配合`SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`和`SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`使用，预先测量好所需的空间，从而提供稳定的视图变化。
#### 注意
* 不能单独使用`SYSTEM_UI_FLAG_LAYOUT_STABLE`，这样从造成界面有为系统UI预留白色空间
* 如果不使用`SYSTEM_UI_FLAG_IMMERSIVE_STICKY`或`SYSTEM_UI_FLAG_IMMERSIVE`，而隐藏系统UI，在点击任意位置都会让系统UI重现
* 使用`SYSTEM_UI_FLAG_IMMERSIVE_STICKY`，不使用`SYSTEM_UI_FLAG_LAYOUT_STABLE`，也不会引起屏幕跳跃，因为粘性沉浸只是暂时重现，而不会清除标记
* 设置`statusBarColor`或`navigationBarColor`为透明后，需要同时设置`SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`或`SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`，不然会有白色空间
* 需要自定义系统UI，`windowDrawsSystemBarBackgrounds`必须为`true`




