> 基于 Android-26 
>
> android.support.v4.widget.NestedScrollView



## Touch 处理

``` java
	@Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        final int action = ev.getAction();
        if ((action == MotionEvent.ACTION_MOVE) && (mIsBeingDragged)) {
          	//快捷处理
            return true;
        }

        switch (action & MotionEventCompat.ACTION_MASK) {
            case MotionEvent.ACTION_MOVE: {
                final int activePointerId = mActivePointerId;
                if (activePointerId == INVALID_POINTER) {
                    // If we don't have a valid id, the touch down wasn't on content.
                    break;
                }

                final int pointerIndex = ev.findPointerIndex(activePointerId);
                if (pointerIndex == -1) {
                    Log.e(TAG, "Invalid pointerId=" + activePointerId
                            + " in onInterceptTouchEvent");
                    break;
                }

                final int y = (int) ev.getY(pointerIndex);
                final int yDiff = Math.abs(y - mLastMotionY);
                if (yDiff > mTouchSlop
                        && (getNestedScrollAxes() & ViewCompat.SCROLL_AXIS_VERTICAL) == 0) {
                  	// 滚动距离超过 TouchSlop，同时垂直滚动
                    mIsBeingDragged = true;
                    mLastMotionY = y;
                    initVelocityTrackerIfNotExists();
                    mVelocityTracker.addMovement(ev);
                    mNestedYOffset = 0;
                    final ViewParent parent = getParent();
                    if (parent != null) {
                        parent.requestDisallowInterceptTouchEvent(true);
                    }
                }
                break;
            }

            case MotionEvent.ACTION_DOWN: {
                final int y = (int) ev.getY();
                if (!inChild((int) ev.getX(), (int) y)) {
                    mIsBeingDragged = false;
                    recycleVelocityTracker();
                    break;
                }

                /*
                 * Remember location of down touch.
                 * ACTION_DOWN always refers to pointer index 0.
                 */
                mLastMotionY = y;
                mActivePointerId = ev.getPointerId(0);

                initOrResetVelocityTracker();
                mVelocityTracker.addMovement(ev);
                //如果 fling 还没结束，用户 DOWN，则 mIsBeingDragged = true
              	//先调用 computeScrollOffset，再调用 isFinished 判断是否 fling 结束
                mScroller.computeScrollOffset();
                mIsBeingDragged = !mScroller.isFinished();
              	startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL, ViewCompat.TYPE_TOUCH);
                //startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL);
                break;
            }

            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP:
                /* Release the drag */
                mIsBeingDragged = false;
                mActivePointerId = INVALID_POINTER;
                recycleVelocityTracker();
                if (mScroller.springBack(getScrollX(), getScrollY(), 0, 0, 0, getScrollRange())) {
                    ViewCompat.postInvalidateOnAnimation(this);
                }
            	stopNestedScroll(ViewCompat.TYPE_TOUCH);
                //stopNestedScroll();
                break;
            case MotionEventCompat.ACTION_POINTER_UP:
            	//处理单个手指抬起的情况
                onSecondaryPointerUp(ev);
                break;
        }

        /*
        * The only time we want to intercept motion events is if we are in the
        * drag mode.
        */
        return mIsBeingDragged;
    }
```

``` java
	@Override
    public boolean onTouchEvent(MotionEvent ev) {
        initVelocityTrackerIfNotExists();

        MotionEvent vtev = MotionEvent.obtain(ev);

        final int actionMasked = MotionEventCompat.getActionMasked(ev);
			
      	//mNestedYOffset 记录被 parent view 嵌套滚动处理后的位移
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            mNestedYOffset = 0;
        }
      	//跟随运动事件滚动，纠正位移
        vtev.offsetLocation(0, mNestedYOffset);

        switch (actionMasked) {
            case MotionEvent.ACTION_DOWN: {
                if (getChildCount() == 0) {
                    return false;
                }
              	//fling 没结束，同时用户 DOWN
                if ((mIsBeingDragged = !mScroller.isFinished())) {
                    final ViewParent parent = getParent();
                    if (parent != null) {
                        parent.requestDisallowInterceptTouchEvent(true);
                    }
                }

                /*
                 * If being flinged and user touches, stop the fling. isFinished
                 * will be false if being flinged.
                 */
                if (!mScroller.isFinished()) {
                  	//停止 fling
                    mScroller.abortAnimation();
                }

                // Remember where the motion event started
                mLastMotionY = (int) ev.getY();
                mActivePointerId = ev.getPointerId(0);
              	startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL, ViewCompat.TYPE_TOUCH);
                //startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL);
                break;
            }
            case MotionEvent.ACTION_MOVE:
                final int activePointerIndex = ev.findPointerIndex(mActivePointerId);
                if (activePointerIndex == -1) {
                    Log.e(TAG, "Invalid pointerId=" + mActivePointerId + " in onTouchEvent");
                    break;
                }

                final int y = (int) ev.getY(activePointerIndex);
                int deltaY = mLastMotionY - y;
                /*if (dispatchNestedPreScroll(0, deltaY, mScrollConsumed, mScrollOffset)) {
                    deltaY -= mScrollConsumed[1];
                    vtev.offsetLocation(0, mScrollOffset[1]);
                    mNestedYOffset += mScrollOffset[1];
                }*/
            	if (dispatchNestedPreScroll(0, deltaY, mScrollConsumed, mScrollOffset,
                                 ViewCompat.TYPE_TOUCH)) {
                  	deltaY -= mScrollConsumed[1];
                  	vtev.offsetLocation(0, mScrollOffset[1]);
                  	mNestedYOffset += mScrollOffset[1];
                }
                if (!mIsBeingDragged && Math.abs(deltaY) > mTouchSlop) {
                    final ViewParent parent = getParent();
                    if (parent != null) {
                        parent.requestDisallowInterceptTouchEvent(true);
                    }
                    mIsBeingDragged = true;
                    if (deltaY > 0) {
                        deltaY -= mTouchSlop;
                    } else {
                        deltaY += mTouchSlop;
                    }
                }
                if (mIsBeingDragged) {
                    //跟随运动事件滚动
                    mLastMotionY = y - mScrollOffset[1];

                    final int oldY = getScrollY();
                    final int range = getScrollRange();
                    final int overscrollMode = getOverScrollMode();
                    boolean canOverscroll = overscrollMode == View.OVER_SCROLL_ALWAYS
                            || (overscrollMode == View.OVER_SCROLL_IF_CONTENT_SCROLLS && range > 0);

                    // Calling overScrollByCompat will call onOverScrolled, which
                    // calls onScrollChanged if applicable.
                    if (overScrollByCompat(0, deltaY, 0, getScrollY(), 0, range, 0,
                            0, true) && !hasNestedScrollingParent()) {
                        // Break our velocity if we hit a scroll barrier.
                        mVelocityTracker.clear();
                    }

                    final int scrolledDeltaY = getScrollY() - oldY;
                    final int unconsumedY = deltaY - scrolledDeltaY;
                    /*if (dispatchNestedScroll(0, scrolledDeltaY, 0, unconsumedY, mScrollOffset)) {*/
                  	if (dispatchNestedScroll(0, scrolledDeltaY, 0, unconsumedY, mScrollOffset, ViewCompat.TYPE_TOUCH)) {
                              
                      	//如果分发给 parent view
                        mLastMotionY -= mScrollOffset[1];
                        vtev.offsetLocation(0, mScrollOffset[1]);
                        mNestedYOffset += mScrollOffset[1];
                    } else if (canOverscroll) {
                      	//边界阴影
                        ensureGlows();
                        final int pulledToY = oldY + deltaY;
                        if (pulledToY < 0) {
                            mEdgeGlowTop.onPull((float) deltaY / getHeight(),
                                    ev.getX(activePointerIndex) / getWidth());
                            if (!mEdgeGlowBottom.isFinished()) {
                                mEdgeGlowBottom.onRelease();
                            }
                        } else if (pulledToY > range) {
                            mEdgeGlowBottom.onPull((float) deltaY / getHeight(),
                                    1.f - ev.getX(activePointerIndex)
                                            / getWidth());
                            if (!mEdgeGlowTop.isFinished()) {
                                mEdgeGlowTop.onRelease();
                            }
                        }
                        if (mEdgeGlowTop != null
                                && (!mEdgeGlowTop.isFinished() || !mEdgeGlowBottom.isFinished())) {
                            ViewCompat.postInvalidateOnAnimation(this);
                        }
                    }
                }
                break;
            case MotionEvent.ACTION_UP:
                if (mIsBeingDragged) {
                  	//fling 处理
                    final VelocityTracker velocityTracker = mVelocityTracker;
                    velocityTracker.computeCurrentVelocity(1000, mMaximumVelocity);
                    int initialVelocity = (int) VelocityTrackerCompat.getYVelocity(velocityTracker,
                            mActivePointerId);

                    if ((Math.abs(initialVelocity) > mMinimumVelocity)) {
                        flingWithNestedDispatch(-initialVelocity);
                    } else if (mScroller.springBack(getScrollX(), getScrollY(), 0, 0, 0,
                            getScrollRange())) {
                      	//如果加速度不够 fling，则 spring back 复位
                        ViewCompat.postInvalidateOnAnimation(this);
                    }
                }
                mActivePointerId = INVALID_POINTER;
                endDrag();
                break;
            case MotionEvent.ACTION_CANCEL:
                if (mIsBeingDragged && getChildCount() > 0) {
                    if (mScroller.springBack(getScrollX(), getScrollY(), 0, 0, 0,
                            getScrollRange())) {
                        ViewCompat.postInvalidateOnAnimation(this);
                    }
                }
                mActivePointerId = INVALID_POINTER;
                endDrag();
                break;
            case MotionEventCompat.ACTION_POINTER_DOWN: {
              	//新的手指按下，替换为当前活动的
                final int index = MotionEventCompat.getActionIndex(ev);
                mLastMotionY = (int) ev.getY(index);
                mActivePointerId = ev.getPointerId(index);
                break;
            }
            case MotionEventCompat.ACTION_POINTER_UP:
                onSecondaryPointerUp(ev);
                mLastMotionY = (int) ev.getY(ev.findPointerIndex(mActivePointerId));
                break;
        }

        if (mVelocityTracker != null) {
            mVelocityTracker.addMovement(vtev);
        }
        vtev.recycle();
        return true;
    }
```

``` java
//注意，调用 flingWithNestedDispatch 是 -velocityY，所有 velocityY > 0 是向上加速度，< 0 则是向下加速度
private void flingWithNestedDispatch(int velocityY) {
        final int scrollY = getScrollY();
  		//scrollY > 0 已经向下滑动一段距离，并且向上加速度
  		//scrollY < getScrollRange 还在滑动范围内，并且向下加速度
        final boolean canFling = (scrollY > 0 || velocityY > 0)
                && (scrollY < getScrollRange() || velocityY < 0);
  		//分发预 Fling
        if (!dispatchNestedPreFling(0, velocityY)) {
          	//父视图不消费 Fling，子视图处理，并且调用 dispatchNestedFling
            dispatchNestedFling(0, velocityY, canFling);
            fling(velocityY);
        }
    }
```



``` java
public void fling(int velocityY) {
        if (getChildCount() > 0) {
          	//开始嵌套滚动，并且 TYPE_NON_TOUCH
            startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL, ViewCompat.TYPE_NON_TOUCH);
            mScroller.fling(getScrollX(), getScrollY(), // start
                    0, velocityY, // velocities
                    0, 0, // x
                    Integer.MIN_VALUE, Integer.MAX_VALUE, // y
                    0, 0); // overscroll
            mLastScrollerY = getScrollY();
            ViewCompat.postInvalidateOnAnimation(this);
        }
    }
```

``` java
@Override
    public void computeScroll() {
        if (mScroller.computeScrollOffset()) {
            final int x = mScroller.getCurrX();
            final int y = mScroller.getCurrY();

            int dy = y - mLastScrollerY;

            // Dispatch up to parent
          	//分发预滚动 ViewCompat.TYPE_NON_TOUCH
            if (dispatchNestedPreScroll(0, dy, mScrollConsumed, null, ViewCompat.TYPE_NON_TOUCH)) {
                dy -= mScrollConsumed[1];
            }

            if (dy != 0) {
                final int range = getScrollRange();
                final int oldScrollY = getScrollY();

                overScrollByCompat(0, dy, getScrollX(), oldScrollY, 0, range, 0, 0, false);

                final int scrolledDeltaY = getScrollY() - oldScrollY;
                final int unconsumedY = dy - scrolledDeltaY;

                if (!dispatchNestedScroll(0, scrolledDeltaY, 0, unconsumedY, null,
                        ViewCompat.TYPE_NON_TOUCH)) {
                  	//如果分发 TYPE_NON_TOUCH 失败了，交由边缘效果处理
                    final int mode = getOverScrollMode();
                    final boolean canOverscroll = mode == OVER_SCROLL_ALWAYS
                            || (mode == OVER_SCROLL_IF_CONTENT_SCROLLS && range > 0);
                    if (canOverscroll) {
                        ensureGlows();
                        if (y <= 0 && oldScrollY > 0) {
                            mEdgeGlowTop.onAbsorb((int) mScroller.getCurrVelocity());
                        } else if (y >= range && oldScrollY < range) {
                            mEdgeGlowBottom.onAbsorb((int) mScroller.getCurrVelocity());
                        }
                    }
                }
            }

            // Finally update the scroll positions and post an invalidation
            mLastScrollerY = y;
            ViewCompat.postInvalidateOnAnimation(this);
        } else {
            // We can't scroll any more, so stop any indirect scrolling
          	//如果存在响应 TYPE_NON_TOUCH，通知结束
            if (hasNestedScrollingParent(ViewCompat.TYPE_NON_TOUCH)) {
                stopNestedScroll(ViewCompat.TYPE_NON_TOUCH);
            }
            // and reset the scroller y
            mLastScrollerY = 0;
        }
    }
```

