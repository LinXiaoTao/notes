* 当 AppBarLayout 设置 `android:fitsSystemWindows="true"` 时，会滚动范围的计算：

  ``` java
  WindowInsetsCompat onWindowInsetChanged(final WindowInsetsCompat insets) {
          WindowInsetsCompat newInsets = null;

          if (ViewCompat.getFitsSystemWindows(this)) {
              // If we're set to fit system windows, keep the insets
            	//只有 fitsSystemWindows = true，才会保存
              newInsets = insets;
          }

          // If our insets have changed, keep them and invalidate the scroll ranges...
          if (!ObjectsCompat.equals(mLastInsets, newInsets)) {
              mLastInsets = newInsets;
              invalidateScrollRanges();
          }

          return insets;
  }

  final int getTopInset() {
          return mLastInsets != null ? mLastInsets.getSystemWindowInsetTop() : 0;
  }

  int getDownNestedPreScrollRange() {
    //省略
    range += childHeight - getTopInset();
  }

  int getDownNestedScrollRange() {
    //省略
    range -= ViewCompat.getMinimumHeight(child) + getTopInset();
  }
  ```

  ​