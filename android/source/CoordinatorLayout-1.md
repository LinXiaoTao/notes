### WindowInsets

CoordinatorLayout 会更改默认的 WindowInsets 处理，将 WindowInsets 进行分发。

``` java
private void setupForInsets() {
        if (Build.VERSION.SDK_INT < 21) {
            return;
        }
		
        if (ViewCompat.getFitsSystemWindows(this)) {
            if (mApplyWindowInsetsListener == null) {
                mApplyWindowInsetsListener =
                        new android.support.v4.view.OnApplyWindowInsetsListener() {
                            @Override
                            public WindowInsetsCompat onApplyWindowInsets(View v,
                                    WindowInsetsCompat insets) {
                                return setWindowInsets(insets);
                            }
                        };
            }
            // First apply the insets listener
            ViewCompat.setOnApplyWindowInsetsListener(this, mApplyWindowInsetsListener);

            //设置 Layout FULLSCREEN，从而获取 WindowInsets 的处理
            setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                    | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
        } else {
            ViewCompat.setOnApplyWindowInsetsListener(this, null);
        }
    }

final WindowInsetsCompat setWindowInsets(WindowInsetsCompat insets) {
        if (!objectEquals(mLastInsets, insets)) {
            mLastInsets = insets;
            mDrawStatusBarBackground = insets != null && insets.getSystemWindowInsetTop() > 0;
            setWillNotDraw(!mDrawStatusBarBackground && getBackground() == null);

            //将 WindowInset 优先分发给 Behaviors
            insets = dispatchApplyWindowInsetsToBehaviors(insets);
            requestLayout();
        }
  		//如果 WindowInset 被 Behaviors 消费后，后续默认的分发流程就不会继续进行了
        return insets;
    }


 private WindowInsetsCompat dispatchApplyWindowInsetsToBehaviors(WindowInsetsCompat insets) {
        if (insets.isConsumed()) {
            return insets;
        }

        for (int i = 0, z = getChildCount(); i < z; i++) {
            final View child = getChildAt(i);
            if (ViewCompat.getFitsSystemWindows(child)) {
                final LayoutParams lp = (LayoutParams) child.getLayoutParams();
                final Behavior b = lp.getBehavior();

                if (b != null) {
                    insets = b.onApplyWindowInsets(this, child, insets);
                    if (insets.isConsumed()) {
                       //当 WindowInset 被消费后，停止分发
                        break;
                    }
                }
            }
        }

        return insets;
    }
```



### StatusBarBackgroup

CoordinatorLayout 默认设置全屏显示，通过下面代码绘制自定义的状态栏：

``` java
@Override
public void onDraw(Canvas c) {
    super.onDraw(c);
    if (mDrawStatusBarBackground && mStatusBarBackground != null) {
        final int inset = mLastInsets != null ? mLastInsets.getSystemWindowInsetTop() : 0;
         if (inset > 0) {
           	//绘制一个和状态栏高度一致的矩形
            mStatusBarBackground.setBounds(0, 0, getWidth(), inset);
            mStatusBarBackground.draw(c);
          }
     }
}
```



### View Changes

这里主要是回调：`onDependentViewRemoved` 和 `onDependentViewChanged` 这两个方法。

``` java
//有三种情况会调用这个方法：
//1.View 被删除时，EVENT_VIEW_REMOVED
//2.View 绘制之前，EVENT_PRE_DRAW
final void onChildViewsChanged(@DispatchChangeEvent final int type) {
        final int layoutDirection = ViewCompat.getLayoutDirection(this);
        final int childCount = mDependencySortedChildren.size();
        final Rect inset = acquireTempRect();
        final Rect drawRect = acquireTempRect();
        final Rect lastDrawRect = acquireTempRect();

        for (int i = 0; i < childCount; i++) {
            final View child = mDependencySortedChildren.get(i);
            final LayoutParams lp = (LayoutParams) child.getLayoutParams();
            if (type == EVENT_PRE_DRAW && child.getVisibility() == View.GONE) {
                // Do not try to update GONE child views in pre draw updates.
                continue;
            }

            // Check child views before for anchor
            for (int j = 0; j < i; j++) {
                final View checkChild = mDependencySortedChildren.get(j);

                if (lp.mAnchorDirectChild == checkChild) {
                  	//处理 Anchor 偏移
                    offsetChildToAnchor(child, layoutDirection);
                }
            }

            // Get the current draw rect of the view
            getChildRect(child, true, drawRect);

            // Accumulate inset sizes
          	//处理子视图如何插入 CoordinatorLayout
          	//根据插入方向，计算相对于 CoordinatorLayout 的最大 Left，Right，Top，Bottom
            if (lp.insetEdge != Gravity.NO_GRAVITY && !drawRect.isEmpty()) {
                final int absInsetEdge = GravityCompat.getAbsoluteGravity(
                        lp.insetEdge, layoutDirection);
                switch (absInsetEdge & Gravity.VERTICAL_GRAVITY_MASK) {
                    case Gravity.TOP:
                        inset.top = Math.max(inset.top, drawRect.bottom);
                        break;
                    case Gravity.BOTTOM:
                        inset.bottom = Math.max(inset.bottom, getHeight() - drawRect.top);
                        break;
                }
                switch (absInsetEdge & Gravity.HORIZONTAL_GRAVITY_MASK) {
                    case Gravity.LEFT:
                        inset.left = Math.max(inset.left, drawRect.right);
                        break;
                    case Gravity.RIGHT:
                        inset.right = Math.max(inset.right, getWidth() - drawRect.left);
                        break;
                }
            }

            // Dodge inset edges if necessary
            if (lp.dodgeInsetEdges != Gravity.NO_GRAVITY && child.getVisibility() == View.VISIBLE) {
              	//处理躲避设置的插入方向
                offsetChildByInset(child, inset, layoutDirection);
            }

            if (type != EVENT_VIEW_REMOVED) {
                // Did it change? if not continue
                getLastChildRect(child, lastDrawRect);
                if (lastDrawRect.equals(drawRect)) {
                    continue;
                }
                recordLastChildRect(child, drawRect);
            }

            // Update any behavior-dependent views for the change
            for (int j = i + 1; j < childCount; j++) {
                final View checkChild = mDependencySortedChildren.get(j);
                final LayoutParams checkLp = (LayoutParams) checkChild.getLayoutParams();
                final Behavior b = checkLp.getBehavior();

                if (b != null && b.layoutDependsOn(this, checkChild, child)) {
                    if (type == EVENT_PRE_DRAW && checkLp.getChangedAfterNestedScroll()) {
                        // If this is from a pre-draw and we have already been changed
                        // from a nested scroll, skip the dispatch and reset the flag
                      	//如果当前是预绘制阶段，并且是由 nested scroll 引起的，跳过
                        checkLp.resetChangedAfterNestedScroll();
                        continue;
                    }

                    final boolean handled;
                    switch (type) {
                        case EVENT_VIEW_REMOVED:
                            // EVENT_VIEW_REMOVED means that we need to dispatch
                            // onDependentViewRemoved() instead
                            b.onDependentViewRemoved(this, checkChild, child);
                            handled = true;
                            break;
                        default:
                            // Otherwise we dispatch onDependentViewChanged()
                            handled = b.onDependentViewChanged(this, checkChild, child);
                            break;
                    }

                    if (type == EVENT_NESTED_SCROLL) {
                        // If this is from a nested scroll, set the flag so that we may skip
                        // any resulting onPreDraw dispatch (if needed)
                      	//如果当前是 nested_scroll 阶段，设置标记，用来在预绘制阶段的时候跳过
                        checkLp.setChangedAfterNestedScroll(handled);
                    }
                }
            }
        }

        releaseTempRect(inset);
        releaseTempRect(drawRect);
        releaseTempRect(lastDrawRect);
    }
```



### Anchor

可以通过 `CoordinatorLayout.LayoutParams.setAnchorId(int)` 去设置 AnchorView，可以不是 CoordinatorLayout 的直接子视图，在 `onMeasure()` 中会根据 AnchorId 去寻找对应的 AnchorView 和 mAnchorDirectChild。

``` java
private void resolveAnchorView(final View forChild, final CoordinatorLayout parent) {
            mAnchorView = parent.findViewById(mAnchorId);
            if (mAnchorView != null) {
                if (mAnchorView == parent) {
                  	//不能将 CoordinatorLayout 设置为 AnchorView
                    if (parent.isInEditMode()) {
                        mAnchorView = mAnchorDirectChild = null;
                        return;
                    }
                    throw new IllegalStateException(
                            "View can not be anchored to the the parent CoordinatorLayout");
                }

                View directChild = mAnchorView;
                for (ViewParent p = mAnchorView.getParent();
                        p != parent && p != null;
                        p = p.getParent()) {
                    if (p == forChild) {
                      	//不能将 AnchorView 设置为 ChildView 的子视图
                        if (parent.isInEditMode()) {
                            mAnchorView = mAnchorDirectChild = null;
                            return;
                        }
                        throw new IllegalStateException(
                                "Anchor must not be a descendant of the anchored view");
                    }
                    if (p instanceof View) {
                        directChild = (View) p;
                    }
                }
              	//AnchorDirectChild 是 AnchorView 的父视图，并且是 Coordinator 的直接子视图
                mAnchorDirectChild = directChild;
            } else {
              	//AnchorView 必须存在当前 CoordinatorLayout 中
                if (parent.isInEditMode()) {
                    mAnchorView = mAnchorDirectChild = null;
                    return;
                }
                throw new IllegalStateException("Could not find CoordinatorLayout descendant view"
                        + " with id " + parent.getResources().getResourceName(mAnchorId)
                        + " to anchor view " + forChild);
            }
        }
```



``` java
void offsetChildToAnchor(View child, int layoutDirection) {
        final LayoutParams lp = (LayoutParams) child.getLayoutParams();
        if (lp.mAnchorView != null) {
            final Rect anchorRect = acquireTempRect();
            final Rect childRect = acquireTempRect();
            final Rect desiredChildRect = acquireTempRect();
			
          	//获取 AnchorView 在 CoordinatorLayout 中位置，可以不是直接子视图，注意：相对于 CoordinatorLayout，而不是 AnchorView 的父视图
            getDescendantRect(lp.mAnchorView, anchorRect);
          	//获取 ChildView 在父视图中的位置
          	//ChildView 一定是 CoordinatorLayout 的直接子视图吗？
            getChildRect(child, false, childRect);

            int childWidth = child.getMeasuredWidth();
            int childHeight = child.getMeasuredHeight();
          	//考虑 ChildView 本身的 gravity 和相对于 AnchorView 的 gravity，计算正确的位置 desiredChildRect
            getDesiredAnchoredChildRectWithoutConstraints(child, layoutDirection, anchorRect,
                    desiredChildRect, lp, childWidth, childHeight);
            boolean changed = desiredChildRect.left != childRect.left ||
                    desiredChildRect.top != childRect.top;
          	//约束新的位置 desiredChildRect
            constrainChildRect(lp, desiredChildRect, childWidth, childHeight);

            final int dx = desiredChildRect.left - childRect.left;
            final int dy = desiredChildRect.top - childRect.top;
			
          	//偏移到新的位置
            if (dx != 0) {
                ViewCompat.offsetLeftAndRight(child, dx);
            }
            if (dy != 0) {
                ViewCompat.offsetTopAndBottom(child, dy);
            }

            if (changed) {
              	//ChildView 位置发生改变，重新通知 onDependentViewChanged
                // If we have needed to move, make sure to notify the child's Behavior
                final Behavior b = lp.getBehavior();
                if (b != null) {
                    b.onDependentViewChanged(this, child, lp.mAnchorView);
                }
            }

            releaseTempRect(anchorRect);
            releaseTempRect(childRect);
            releaseTempRect(desiredChildRect);
        }
    }
```

