### InsetEdge

`layout:insetEdge`： Gravity 值。描述 ChildView 如何插入 CoordinatorLayout

`layout:dodgeInsetEdges`：Gravity 值。描述 ChildView 是否躲避 `layout:insetEdge` 设置为该值的 ChildViews。

``` java
//inset 是由 onChildViewsChanged 方法计算的，各个 Gravity 插入的 ChildViews 所需要的最大 left，right，top，bottom 的值，这里可以理解为：各个方向的边距
private void offsetChildByInset(final View child, final Rect inset, final int layoutDirection) {
        if (!ViewCompat.isLaidOut(child)) {
            // The view has not been laid out yet, so we can't obtain its bounds.
            return;
        }

        if (child.getWidth() <= 0 || child.getHeight() <= 0) {
            // Bounds are empty so there is nothing to dodge against, skip...
            return;
        }

        final LayoutParams lp = (LayoutParams) child.getLayoutParams();
        final Behavior behavior = lp.getBehavior();
        final Rect dodgeRect = acquireTempRect();
        final Rect bounds = acquireTempRect();
        bounds.set(child.getLeft(), child.getTop(), child.getRight(), child.getBottom());

  		//是否需要躲避其他插入 CoordinatorLayout 的 ChildViews
        if (behavior != null && behavior.getInsetDodgeRect(this, child, dodgeRect)) {
            // Make sure that the rect is within the view's bounds
            if (!bounds.contains(dodgeRect)) {
                throw new IllegalArgumentException("Rect should be within the child's bounds."
                        + " Rect:" + dodgeRect.toShortString()
                        + " | Bounds:" + bounds.toShortString());
            }
        } else {
            dodgeRect.set(bounds);
        }

        // We can release the bounds rect now
        releaseTempRect(bounds);

        if (dodgeRect.isEmpty()) {
            // Rect is empty so there is nothing to dodge against, skip...
            releaseTempRect(dodgeRect);
            return;
        }

        final int absDodgeInsetEdges = GravityCompat.getAbsoluteGravity(lp.dodgeInsetEdges,
                layoutDirection);

        boolean offsetY = false;
        if ((absDodgeInsetEdges & Gravity.TOP) == Gravity.TOP) {
            int distance = dodgeRect.top - lp.topMargin - lp.mInsetOffsetY;
            if (distance < inset.top) {
                setInsetOffsetY(child, inset.top - distance);
                offsetY = true;
            }
        }
        if ((absDodgeInsetEdges & Gravity.BOTTOM) == Gravity.BOTTOM) {
            int distance = getHeight() - dodgeRect.bottom - lp.bottomMargin + lp.mInsetOffsetY;
            if (distance < inset.bottom) {
                setInsetOffsetY(child, distance - inset.bottom);
                offsetY = true;
            }
        }
        if (!offsetY) {
            setInsetOffsetY(child, 0);
        }

        boolean offsetX = false;
        if ((absDodgeInsetEdges & Gravity.LEFT) == Gravity.LEFT) {
            int distance = dodgeRect.left - lp.leftMargin - lp.mInsetOffsetX;
            if (distance < inset.left) {
                setInsetOffsetX(child, inset.left - distance);
                offsetX = true;
            }
        }
        if ((absDodgeInsetEdges & Gravity.RIGHT) == Gravity.RIGHT) {
            int distance = getWidth() - dodgeRect.right - lp.rightMargin + lp.mInsetOffsetX;
            if (distance < inset.right) {
                setInsetOffsetX(child, distance - inset.right);
                offsetX = true;
            }
        }
        if (!offsetX) {
            setInsetOffsetX(child, 0);
        }

        releaseTempRect(dodgeRect);
    }
```



### Touch

``` java
	@Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        MotionEvent cancelEvent = null;

        final int action = ev.getActionMasked();

        // Make sure we reset in case we had missed a previous important event.
        if (action == MotionEvent.ACTION_DOWN) {
            resetTouchBehaviors();
        }

      	//是否拦截
        final boolean intercepted = performIntercept(ev, TYPE_ON_INTERCEPT);

        if (cancelEvent != null) {
            cancelEvent.recycle();
        }

        if (action == MotionEvent.ACTION_UP || action == MotionEvent.ACTION_CANCEL) {
            resetTouchBehaviors();
        }

        return intercepted;
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        boolean handled = false;
        boolean cancelSuper = false;
        MotionEvent cancelEvent = null;

        final int action = ev.getActionMasked();

        if (mBehaviorTouchView != null || (cancelSuper = performIntercept(ev, TYPE_ON_TOUCH))) {
            // Safe since performIntercept guarantees that
            // mBehaviorTouchView != null if it returns true
            final LayoutParams lp = (LayoutParams) mBehaviorTouchView.getLayoutParams();
            final Behavior b = lp.getBehavior();
            if (b != null) {
              	//分发给 mBehaviorTouchView
                handled = b.onTouchEvent(this, mBehaviorTouchView, ev);
            }
        }

        // Keep the super implementation correct
        if (mBehaviorTouchView == null) {
            handled |= super.onTouchEvent(ev);
        } else if (cancelSuper) {
            if (cancelEvent == null) {
                final long now = SystemClock.uptimeMillis();
                cancelEvent = MotionEvent.obtain(now, now,
                        MotionEvent.ACTION_CANCEL, 0.0f, 0.0f, 0);
            }
            super.onTouchEvent(cancelEvent);
        }

        if (!handled && action == MotionEvent.ACTION_DOWN) {

        }

        if (cancelEvent != null) {
            cancelEvent.recycle();
        }

        if (action == MotionEvent.ACTION_UP || action == MotionEvent.ACTION_CANCEL) {
            resetTouchBehaviors();
        }

        return handled;
    }
```

``` java
//执行拦截
private boolean performIntercept(MotionEvent ev, final int type) {
        boolean intercepted = false;
        boolean newBlock = false;

        MotionEvent cancelEvent = null;

        final int action = ev.getActionMasked();

        final List<View> topmostChildList = mTempList1;
        getTopSortedChildren(topmostChildList);

        // Let topmost child views inspect first
        final int childCount = topmostChildList.size();
        for (int i = 0; i < childCount; i++) {
            final View child = topmostChildList.get(i);
            final LayoutParams lp = (LayoutParams) child.getLayoutParams();
            final Behavior b = lp.getBehavior();

            if ((intercepted || newBlock) && action != MotionEvent.ACTION_DOWN) {
                // Cancel all behaviors beneath the one that intercepted.
                // If the event is "down" then we don't have anything to cancel yet.
              	//如果当前不是 DOWN 事件，并且事件已经被消费或者阻塞，分发 CANCEL 事件
                if (b != null) {
                    if (cancelEvent == null) {
                        final long now = SystemClock.uptimeMillis();
                        cancelEvent = MotionEvent.obtain(now, now,
                                MotionEvent.ACTION_CANCEL, 0.0f, 0.0f, 0);
                    }
                    switch (type) {
                        case TYPE_ON_INTERCEPT:
                            b.onInterceptTouchEvent(this, child, cancelEvent);
                            break;
                        case TYPE_ON_TOUCH:
                            b.onTouchEvent(this, child, cancelEvent);
                            break;
                    }
                }
                continue;
            }

            if (!intercepted && b != null) {
              	//结合 onTouch()，先决条件是 mBehaviorTouchView = null，这样不会出现重复调用
              	// onTouchEvent，在 onTouch 中会调用  b.onTouchEvent()
                switch (type) {
                    case TYPE_ON_INTERCEPT:
                        intercepted = b.onInterceptTouchEvent(this, child, ev);
                        break;
                    case TYPE_ON_TOUCH:
                        intercepted = b.onTouchEvent(this, child, ev);
                        break;
                }
                if (intercepted) {
                    mBehaviorTouchView = child;
                }
            }

            // Don't keep going if we're not allowing interaction below this.
            // Setting newBlock will make sure we cancel the rest of the behaviors.
            final boolean wasBlocking = lp.didBlockInteraction();
            final boolean isBlocking = lp.isBlockingInteractionBelow(this, child);
            newBlock = isBlocking && !wasBlocking;
            if (isBlocking && !newBlock) {
              	//如果当前 ChildView 拦截了后面的 ChildViews 的视角
                // Stop here since we don't have anything more to cancel - we already did
                // when the behavior first started blocking things below this point.
                break;
            }
        }

        topmostChildList.clear();

        return intercepted;
    }
```



### Dependency

在 `onMeasure()` 中调用下面这个方法来构建 ChildViews 的依赖关系：

``` java
private void prepareChildren() {
        mDependencySortedChildren.clear();
        mChildDag.clear();

        for (int i = 0, count = getChildCount(); i < count; i++) {
            final View view = getChildAt(i);

            final LayoutParams lp = getResolvedLayoutParams(view);
          	//根据 anchorId 获取 AnchorView
            lp.findAnchorView(this, view);
			//有向无环图
          	//添加节点
            mChildDag.addNode(view);

            // Now iterate again over the other children, adding any dependencies to the graph
            for (int j = 0; j < count; j++) {
                if (j == i) {
                    continue;
                }
                final View other = getChildAt(j);
              	//存在依赖关系
                if (lp.dependsOn(this, view, other)) {
                    if (!mChildDag.contains(other)) {
                        // Make sure that the other node is added
                      	//添加节点
                        mChildDag.addNode(other);
                    }
                    // Now add the dependency to the graph
                  	//添加边
                    mChildDag.addEdge(other, view);
                }
            }
        }

        // Finally add the sorted graph list to our list
  		//添加有向无环图的拓扑排序，使用 DFS 算法
        mDependencySortedChildren.addAll(mChildDag.getSortedList());
        // We also need to reverse the result since we want the start of the list to contain
        // Views which have no dependencies, then dependent views after that
  		//反向
        Collections.reverse(mDependencySortedChildren);
    }


//是否存在依赖关系
//1.是否为 mAnchorDirectChild，即 AnchorView 的 CoordinatorLayout 直接子视图
//2.是否存在躲避插入 CoordinatorLayout 方向的关系
//3.调用 Behavior.layoutDependsOn
boolean dependsOn(CoordinatorLayout parent, View child, View dependency) {
  	 return dependency == mAnchorDirectChild
       		|| shouldDodge(dependency, ViewCompat.getLayoutDirection(parent))
       		|| (mBehavior != null && mBehavior.layoutDependsOn(parent, child, dependency));
}
```



### Nested Scrolling

``` java
	@Override
    public boolean onStartNestedScroll(View child, View target, int axes, int type) {
        	boolean handled = false;
        	//省略
            final Behavior viewBehavior = lp.getBehavior();
            if (viewBehavior != null) {
              	//分发给 onStartNestedScroll
                final boolean accepted = viewBehavior.onStartNestedScroll(this, view, child,
                        target, axes, type);
                handled |= accepted;
                lp.setNestedScrollAccepted(type, accepted);
            } else {
                lp.setNestedScrollAccepted(type, false);
            }
        }
        return handled;
    }
```

``` java
	@Override
    public void onNestedScrollAccepted(View child, View target, int nestedScrollAxes, int type) {
       		//省略
            final Behavior viewBehavior = lp.getBehavior();
            if (viewBehavior != null) {
              	//分发 onNestedScrollAccepted
                viewBehavior.onNestedScrollAccepted(this, view, child, target,
                        nestedScrollAxes, type);
            }
        }
    }
```

``` java
	@Override
    public void onStopNestedScroll(View target, int type) {
        	//省略
            final Behavior viewBehavior = lp.getBehavior();
            if (viewBehavior != null) {
              	//分发 onStopNestedScroll
                viewBehavior.onStopNestedScroll(this, view, target, type);
            }
            lp.resetNestedScroll(type);
            lp.resetChangedAfterNestedScroll();
        }
        mNestedScrollingTarget = null;
    }
```

``` java
	@Override
    public void onNestedScroll(View target, int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, int type) {
        	//省略
            final Behavior viewBehavior = lp.getBehavior();
            if (viewBehavior != null) {
              	//分发 onNestedScroll
                viewBehavior.onNestedScroll(this, view, target, dxConsumed, dyConsumed,
                        dxUnconsumed, dyUnconsumed, type);
                accepted = true;
            }
        }

        if (accepted) {
          	//EVENT_NESTED_SCROLL
            onChildViewsChanged(EVENT_NESTED_SCROLL);
        }
    }
```

``` java
 	@Override
    public void onNestedPreScroll(View target, int dx, int dy, int[] consumed, int  type) {
        	//省略
            final Behavior viewBehavior = lp.getBehavior();
            if (viewBehavior != null) {
                mTempIntPair[0] = mTempIntPair[1] = 0;
              	//分发 onNestedPreScroll
                viewBehavior.onNestedPreScroll(this, view, target, dx, dy, mTempIntPair, type);
				//取最大值
                xConsumed = dx > 0 ? Math.max(xConsumed, mTempIntPair[0])
                        : Math.min(xConsumed, mTempIntPair[0]);
                yConsumed = dy > 0 ? Math.max(yConsumed, mTempIntPair[1])
                        : Math.min(yConsumed, mTempIntPair[1]);

                accepted = true;
            }
        }

        consumed[0] = xConsumed;
        consumed[1] = yConsumed;

        if (accepted) {
          	//EVENT_NESTED_SCROLL
            onChildViewsChanged(EVENT_NESTED_SCROLL);
        }
    }
```

``` java
	@Override
    public boolean onNestedFling(View target, float velocityX, float velocityY, boolean consumed) {
        	//省略
            final Behavior viewBehavior = lp.getBehavior();
            if (viewBehavior != null) {
              	//分发 onNestedFling
                handled |= viewBehavior.onNestedFling(this, view, target, velocityX, velocityY,
                        consumed);
            }
        }
        if (handled) {
            onChildViewsChanged(EVENT_NESTED_SCROLL);
        }
        return handled;
    }
```

``` java
	@Override
    public boolean onNestedPreFling(View target, float velocityX, float velocityY) {
        	//省略
            final Behavior viewBehavior = lp.getBehavior();
            if (viewBehavior != null) {
              	//分发 onNestedPreFling
                handled |= viewBehavior.onNestedPreFling(this, view, target, velocityX, velocityY);
            }
        }
        return handled;
    }
```



### 其他

``` java
//默认存在 AnchorView 的 ChildView 的 gravity 是 CENTER
private static int resolveAnchoredChildGravity(int gravity) {
  return gravity == Gravity.NO_GRAVITY ? Gravity.CENTER : gravity;
}

//默认 gravity 是 LEFT | TOP
private static int resolveGravity(int gravity) {
  	if ((gravity & Gravity.HORIZONTAL_GRAVITY_MASK) == Gravity.NO_GRAVITY) {
      	gravity |= GravityCompat.START;
    }
  	if ((gravity & Gravity.VERTICAL_GRAVITY_MASK) == Gravity.NO_GRAVITY) {
      	 gravity |= Gravity.TOP;
    }
  	 return gravity;
}
```

``` java
	@Override
    protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
        final LayoutParams lp = (LayoutParams) child.getLayoutParams();
        if (lp.mBehavior != null) {
          	//getScrimOpacity 获取 Scrim 的不透明度，0.f - 1.0f，0 表示透明
          	//blocksInteractionBelow 方法默认返回 getScrimOpacity > 0.f
            final float scrimAlpha = lp.mBehavior.getScrimOpacity(this, child);
            if (scrimAlpha > 0f) {
                if (mScrimPaint == null) {
                    mScrimPaint = new Paint();
                }
              	//Scrim 类似一层遮罩
                mScrimPaint.setColor(lp.mBehavior.getScrimColor(this, child));
                mScrimPaint.setAlpha(MathUtils.clamp(Math.round(255 * scrimAlpha), 0, 255));

                final int saved = canvas.save();
                if (child.isOpaque()) {
                    // If the child is opaque, there is no need to draw behind it so we'll inverse
                    // clip the canvas
                    canvas.clipRect(child.getLeft(), child.getTop(), child.getRight(),
                            child.getBottom(), Region.Op.DIFFERENCE);
                }
                // Now draw the rectangle for the scrim
                canvas.drawRect(getPaddingLeft(), getPaddingTop(),
                        getWidth() - getPaddingRight(), getHeight() - getPaddingBottom(),
                        mScrimPaint);
                canvas.restoreToCount(saved);
            }
        }
        return super.drawChild(canvas, child, drawingTime);
    }
```

