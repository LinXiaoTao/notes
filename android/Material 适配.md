### 知识点

* 关于 `android:theme` ，`app:theme`

  Android 5.0 引入一个全新的特性，允许给 view 设置 theme，这种设置会影响控件及其包含的子控件。

  在 AppCompat v21 里，提供了设置 Toolbar 的主题，使用 `app:theme`。

  而在 22.1.x 中，AppCompat 允许对 Toolbar 使用 `android:theme` 代替 `app:theme` ，它会自动继承父视图的 theme，并且兼容所有 APIv11 以上的设备。

  对于运行 APIv10 甚至更老的设备来说，也可以使用 `android:theme` 属性，不过它不会继承父视图的 theme。

* 关于 `?android:attr` 和 `?attr` 的区别

  完整的表达式：

  ``` xml
  ?[*<package_name>*:][*<resource_type>*/]*<resource_name>*
  ```

  `?attr` 表示引用当前主题中的资源，所以可以省去 package_name

  `?android:attr` 表示引用的是 android 系统中的一些资源。

  某些属性可能当前主题中和 android 系统中都存在，当前主题会覆盖系统中的值。

### Toolbar

* 关于 `app:popupTheme`

  有时候需要 ActionBar 的文字是白的，ActionBar Overflow 弹出的是白底黑字。

  让 ActionBar 文字是白的，对于的 theme 是 Dark。

  让 ActionBar Overflow 弹出的是白底黑字，则需要 Light。

  这时候可以用 `app:popupTheme`

  ``` xml
  <android.support.v7.widget.Toolbar
      android:id="@+id/toolbar"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:background="?attr/colorPrimary"
      app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
      android:minHeight="?attr/actionBarSize" />
  ```

作为 ActionBar 使用：

```java
Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
setSupportActionBar(toolbar);
```

独立使用：

``` java
// Set an OnMenuItemClickListener to handle menu item clicks
    toolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener() {
        @Override
        public boolean onMenuItemClick(MenuItem item) {
            // Handle the menu item
            return true;
        }
    });

    // Inflate a menu to be displayed in the toolbar
    toolbar.inflateMenu(R.menu.your_toolbar_menu);
```

### DrawerLayout

* Drawable 和 Toolbar 配合使用

  ``` xml
  <style name=”MyAppTheme” parent=”@style/Theme.AppCompat.Light.NoActionBar”>
  …
    <!- 这个可要可不要 -->
  <item name=”android:windowDrawsSystemBarBackgrounds”>true</item>
  <item name=”android:windowTranslucentStatus”>true</item>
  …
  </style>
  ```

  ``` xml
  <android.support.v4.widget.DrawerLayout 
   android:id=”@+id/drawerLayout”
   android:layout_width=”match_parent”
   android:layout_height=”match_parent”
   android:fitsSystemWindows=”true”
   >
  <!— The content -->
   <include layout=”@layout/main_activity_content”/>
  <!— The drawer -->
   <LinearLayout
   android:id=”@+id/drawerPanel”
   android:layout_width=”320dp”
   android:layout_height=”match_parent”
   android:orientation=”vertical”
   android:layout_gravity=”left|start”
   android:fitsSystemWindows=”true”
   android:layout_marginTop="-25dp"
   >
   <!- drawer 内容如果想延伸到状态栏，而且自动适应状态栏高度，需设置  fitsSystemWindows = true -->
     <!- layout_marginTop = -25dp 这个可要可不要，设置了可以弥补适应状态栏高度所设置的 padding ->
  </android.support.v4.widget.DrawerLayout>
  ```

  ``` java
  drawerLayout = (DrawerLayout)  
        findViewById(R.id.drawerLayout);  
     drawerLayout.setStatusBarBackground(R.color.primary_dark);
  //在 style 中设置了 windowTranslucentStatus 或 statusBarColor = transparent ，那么只能在 DrawerLayout 中设置状态栏颜色，注意 DrawerLayout 需设置 fitsSystemWindows = true，而且状态栏颜色必须是透明的
  ```

* DrawerLayout 对 fitsSystemWindows = true 的处理

  ``` java
  if (ViewCompat.getFitsSystemWindows(this)) {
    			//如果对 DrawerLayout 设置了 setFitsSystemWindows(true)，则会自动设置 FULLSCREEN，且设置默认状态栏颜色
              IMPL.configureApplyInsets(this);
              mStatusBarBackground = IMPL.getDefaultStatusBarBackground(context);
          }

   public static void configureApplyInsets(View drawerLayout) {
          if (drawerLayout instanceof DrawerLayoutImpl) {
            	//这里设置了 OnApplyWindowInsetsListener ，做特殊处理，默认为设置 padding
              drawerLayout.setOnApplyWindowInsetsListener(new InsetsListener());
              drawerLayout.setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                      | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
          }
      }
  ```

  ```java
  if (applyInsets) {
    				// DrawerLayout 会对子视图的的 FitsSystemWindow 进行特殊处理
                  final int cgrav = GravityCompat.getAbsoluteGravity(lp.gravity, layoutDirection);
                  if (ViewCompat.getFitsSystemWindows(child)) {
                    	//如果子视图也设置了 setFitsSystemWindows(true)，则继续分发给子视图
                      IMPL.dispatchChildInsets(child, mLastInsets, cgrav);
                  } else {
                    	//否则，设置它的 margin
                      IMPL.applyMarginInsets(lp, mLastInsets, cgrav);
                  }
              }

  public static void dispatchChildInsets(View child, Object insets, int gravity) {
          WindowInsets wi = (WindowInsets) insets;
          if (gravity == Gravity.LEFT) {
              wi = wi.replaceSystemWindowInsets(wi.getSystemWindowInsetLeft(),
                      wi.getSystemWindowInsetTop(), 0, wi.getSystemWindowInsetBottom());
          } else if (gravity == Gravity.RIGHT) {
              wi = wi.replaceSystemWindowInsets(0, wi.getSystemWindowInsetTop(),
                      wi.getSystemWindowInsetRight(), wi.getSystemWindowInsetBottom());
          }
          child.dispatchApplyWindowInsets(wi);
      }


   public static void applyMarginInsets(ViewGroup.MarginLayoutParams lp, Object insets,
              int gravity) {
          WindowInsets wi = (WindowInsets) insets;
          if (gravity == Gravity.LEFT) {
              wi = wi.replaceSystemWindowInsets(wi.getSystemWindowInsetLeft(),
                      wi.getSystemWindowInsetTop(), 0, wi.getSystemWindowInsetBottom());
          } else if (gravity == Gravity.RIGHT) {
              wi = wi.replaceSystemWindowInsets(0, wi.getSystemWindowInsetTop(),
                      wi.getSystemWindowInsetRight(), wi.getSystemWindowInsetBottom());
          }
          lp.leftMargin = wi.getSystemWindowInsetLeft();
          lp.topMargin = wi.getSystemWindowInsetTop();
          lp.rightMargin = wi.getSystemWindowInsetRight();
          lp.bottomMargin = wi.getSystemWindowInsetBottom();
      }
  ```

  ​