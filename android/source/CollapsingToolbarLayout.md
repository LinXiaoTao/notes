* CollapsingToolbarLayout 要和 `AppBarLayout` 配合使用

  ``` java
  	@Override
      protected void onAttachedToWindow() {
          super.onAttachedToWindow();

          // Add an OnOffsetChangedListener if possible
          final ViewParent parent = getParent();
          if (parent instanceof AppBarLayout) {
              // Copy over from the ABL whether we should fit system windows
              ViewCompat.setFitsSystemWindows(this, ViewCompat.getFitsSystemWindows((View) parent));

              if (mOnOffsetChangedListener == null) {
                  mOnOffsetChangedListener = new OffsetUpdateListener();
              }
            //addOnOffsetChangedListener
          ((AppBarLayout)parent).addOnOffsetChangedListener(mOnOffsetChangedListener);

              // We're attached, so lets request an inset dispatch
              ViewCompat.requestApplyInsets(this);
          }
      }
  ```

* CollapsingToolbarLayout 一般使用 `app:layout_scrollFlags="scroll|exitUntilCollapsed"` 来实现滚动后折叠的效果，而 `AppBarLayout` 折叠的高度是按子视图的 `getMinimumHeight()` 确定的。CollapsingToolbarLayout 根据 Toolbar 的高度来设置最小高度

  ``` java
  if (mToolbar != null) {
              if (mCollapsingTitleEnabled && TextUtils.isEmpty(mCollapsingTextHelper.getText())) {
                  // If we do not currently have a title, try and grab it from the Toolbar
                  mCollapsingTextHelper.setText(mToolbar.getTitle());
              }
              if (mToolbarDirectChild == null || mToolbarDirectChild == this) {
                  setMinimumHeight(getHeightWithMargins(mToolbar));
              } else {
                  setMinimumHeight(getHeightWithMargins(mToolbarDirectChild));
              }
          }
  ```

* CollapsingToolbarLayout 可以设置三种折叠模式：`COLLAPSE_MODE_OFF`，`COLLAPSE_MODE_PIN` 和 `COLLAPSE_MODE_PARALLAX`。

  ```
  COLLAPSE_MODE_OFF：默认行为，跟随父视图偏移
  COLLAPSE_MODE_PIN：开始固定在原先位置，直到父视图偏移到该视图的底部，然后跟随父视图偏移
  COLLAPSE_MODE_PARALLAX：跟随父视图偏移，但是有个与父视图相反方向的偏移，形成视觉差
  ```

  ``` java
  private class OffsetUpdateListener implements AppBarLayout.OnOffsetChangedListener {
          OffsetUpdateListener() {
          }

          @Override
          public void onOffsetChanged(AppBarLayout layout, int verticalOffset) {
              mCurrentOffset = verticalOffset;

              final int insetTop = mLastInsets != null ? mLastInsets.getSystemWindowInsetTop() : 0;

              for (int i = 0, z = getChildCount(); i < z; i++) {
                  final View child = getChildAt(i);
                  final LayoutParams lp = (LayoutParams) child.getLayoutParams();
                  final ViewOffsetHelper offsetHelper = getViewOffsetHelper(child);

                  switch (lp.mCollapseMode) {
                      case LayoutParams.COLLAPSE_MODE_PIN:
                      	//反方向偏移到一个临界值
                          offsetHelper.setTopAndBottomOffset(MathUtils.clamp(
                                  -verticalOffset, 0, getMaxOffsetForPinChild(child)));
                          break;
                      case LayoutParams.COLLAPSE_MODE_PARALLAX:
                      	//反方向的偏移
                          offsetHelper.setTopAndBottomOffset(
                                  Math.round(-verticalOffset * lp.mParallaxMult));
                          break;
                  }
              }

              // Show or hide the scrims if needed
              updateScrimVisibility();

              if (mStatusBarScrim != null && insetTop > 0) {
                  ViewCompat.postInvalidateOnAnimation(CollapsingToolbarLayout.this);
              }

              // Update the collapsing text's fraction
              final int expandRange = getHeight() - ViewCompat.getMinimumHeight(
                      CollapsingToolbarLayout.this) - insetTop;
              mCollapsingTextHelper.setExpansionFraction(
                      Math.abs(verticalOffset) / (float) expandRange);
          }
      }
  ```

* CollapsingToolbarLayout 会将没有设置的 `android:fitsSystemWindows="true"` 的子视图偏移到状态栏的下面，不会被状态栏遮挡：

  ``` java
  if (mLastInsets != null) {
              // Shift down any views which are not set to fit system windows
              final int insetTop = mLastInsets.getSystemWindowInsetTop();
              for (int i = 0, z = getChildCount(); i < z; i++) {
                  final View child = getChildAt(i);
                  if (!ViewCompat.getFitsSystemWindows(child)) {
                      if (child.getTop() < insetTop) {
                          // If the child isn't set to fit system windows but is drawing within
                          // the inset offset it down
                          ViewCompat.offsetTopAndBottom(child, insetTop);
                      }
                  }
              }
          }
  ```

  同时，如果设置了 StatusBarScim 的话，就会绘制 StatusBarScim：

  ``` java
  if (mStatusBarScrim != null && mScrimAlpha > 0) {
              final int topInset = mLastInsets != null ? mLastInsets.getSystemWindowInsetTop() : 0;
              if (topInset > 0) {
                  mStatusBarScrim.setBounds(0, -mCurrentOffset, getWidth(),
                          topInset - mCurrentOffset);
                  mStatusBarScrim.mutate().setAlpha(mScrimAlpha);
                  mStatusBarScrim.draw(canvas);
              }
          }
  ```

* StatusBarScim 是当 `android:fitsSystemWindows="true` 时，绘制在状态栏的 Drawable。默认是 但状态栏颜色的 `ColorDrawable` 。跟随 CollapsingToolbarLayout 的偏移设置透明度。

* ContentScim 是绘制在 CollapsingToolbarLayout 的 Drawable，默认是 null。跟随 CollapsingToolbarLayout 的偏移设置透明度。

* CollapsingToolbarLayout 设置为 `android:fitsSystemWindows="true"` 会影响在 `AppBarLayout` 中的滚动范围

  > AppBarLayout.interpolateOffset

  ``` java
  if (ViewCompat.getFitsSystemWindows(child)) {
        childScrollableHeight -= layout.getTopInset();
  }
  ```

  ​

* 如果 CollapsingToolbarLayout 的 Parent View 是 `AppBarLayout`，会跟随 `AppBarLayout` 的 `fitsSystemWindows`：

  ``` java
  @Override
      protected void onAttachedToWindow() {
          super.onAttachedToWindow();

          // Add an OnOffsetChangedListener if possible
          final ViewParent parent = getParent();
          if (parent instanceof AppBarLayout) {
              // Copy over from the ABL whether we should fit system windows
              ViewCompat.setFitsSystemWindows(this, ViewCompat.getFitsSystemWindows((View) parent));

              if (mOnOffsetChangedListener == null) {
                  mOnOffsetChangedListener = new OffsetUpdateListener();
              }
              ((AppBarLayout) parent).addOnOffsetChangedListener(mOnOffsetChangedListener);

              // We're attached, so lets request an inset dispatch
              ViewCompat.requestApplyInsets(this);
          }
      }
  ```

* CollapsingToolbarLayout 默认会消费 WindowInsets：

  ``` java
  WindowInsetsCompat onWindowInsetChanged(final WindowInsetsCompat insets) {
          WindowInsetsCompat newInsets = null;

          if (ViewCompat.getFitsSystemWindows(this)) {
              // If we're set to fit system windows, keep the insets
              newInsets = insets;
          }

          // If our insets have changed, keep them and invalidate the scroll ranges...
          if (!ObjectsCompat.equals(mLastInsets, newInsets)) {
              mLastInsets = newInsets;
              requestLayout();
          }

          // Consume the insets. This is done so that child views with fitSystemWindows=true do not
          // get the default padding functionality from View
    		//消费 WindowInsets
          return insets.consumeSystemWindowInsets();
      }
  ```

  ​