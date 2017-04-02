``` java
 public static ViewDragHelper create(ViewGroup forParent, Callback cb) {
        return new ViewDragHelper(forParent.getContext(), forParent, cb);
    }
    
private ViewDragHelper(Context context, ViewGroup forParent, Callback cb) {
        if (forParent == null) {
            throw new IllegalArgumentException("Parent view may not be null");
        }
        if (cb == null) {
            throw new IllegalArgumentException("Callback may not be null");
        }

	//保存父视图
        mParentView = forParent;
        //保存回调方法
        mCallback = cb;

	
        final ViewConfiguration vc = ViewConfiguration.get(context);
        final float density = context.getResources().getDisplayMetrics().density;
        //获取边缘大小
        mEdgeSize = (int) (EDGE_SIZE * density + 0.5f);
	
	//获取系统定义常量
        mTouchSlop = vc.getScaledTouchSlop();
        mMaxVelocity = vc.getScaledMaximumFlingVelocity();
        mMinVelocity = vc.getScaledMinimumFlingVelocity();
        //获取Scroller
        mScroller = ScrollerCompat.create(context, sInterpolator);
    }
    
public boolean shouldInterceptTouchEvent(MotionEvent ev) {
	//多指处理
        final int action = MotionEventCompat.getActionMasked(ev);
        final int actionIndex = MotionEventCompat.getActionIndex(ev);

        if (action == MotionEvent.ACTION_DOWN) {
            // Reset things for a new event stream, just in case we didn't get
            // the whole previous stream.
            //重置事件处理
            cancel();
        }

	//速度监视
        if (mVelocityTracker == null) {
            mVelocityTracker = VelocityTracker.obtain();
        }
        mVelocityTracker.addMovement(ev);

        switch (action) {
            case MotionEvent.ACTION_DOWN: {
                final float x = ev.getX();
                final float y = ev.getY();
                final int pointerId = ev.getPointerId(0);
                //保存初始触摸实际信息
                saveInitialMotion(x, y, pointerId);
		//获取当前触摸视图
                final View toCapture = findTopChildUnder((int) x, (int) y);

                // Catch a settling view if possible.
                //捕获一个因为惯性或者其他预定义的非交互式运动的视图
                //注意:比如调用settleCapturedViewAt或smoothSlideViewTo，视图会以定义的速率滚动
                if (toCapture == mCapturedView && mDragState == STATE_SETTLING) {
                    tryCaptureViewForDrag(toCapture, pointerId);
                }

		//触摸边界回调
                final int edgesTouched = mInitialEdgesTouched[pointerId];
                if ((edgesTouched & mTrackingEdges) != 0) {
                    mCallback.onEdgeTouched(edgesTouched & mTrackingEdges, pointerId);
                }
                break;
            }

            case MotionEventCompat.ACTION_POINTER_DOWN: {
            	//处理非主指针按下的情况，即又按下一个手指
                final int pointerId = ev.getPointerId(actionIndex);
                final float x = ev.getX(actionIndex);
                final float y = ev.getY(actionIndex);
		//保存最后按下的指针信息
                saveInitialMotion(x, y, pointerId);

                // A ViewDragHelper can only manipulate one view at a time.
                if (mDragState == STATE_IDLE) {
                    //当前拖拽状态为初始状态，即没有处理任务，处理边界情况
                    final int edgesTouched = mInitialEdgesTouched[pointerId];
                    if ((edgesTouched & mTrackingEdges) != 0) {
                        mCallback.onEdgeTouched(edgesTouched & mTrackingEdges, pointerId);
                    }
                } else if (mDragState == STATE_SETTLING) {
                    // Catch a settling view if possible.
                    //捕获一个因为惯性或者其他预定义的非交互式运动的视图
                    final View toCapture = findTopChildUnder((int) x, (int) y);
                    if (toCapture == mCapturedView) {
                        tryCaptureViewForDrag(toCapture, pointerId);
                    }
                }
                break;
            }

            case MotionEvent.ACTION_MOVE: {
            	//检查初始按下值
                if (mInitialMotionX == null || mInitialMotionY == null) break;

                // First to cross a touch slop over a draggable view wins. Also report edge drags.
                final int pointerCount = ev.getPointerCount();
                for (int i = 0; i < pointerCount; i++) {
                    final int pointerId = ev.getPointerId(i);

                    // If pointer is invalid then skip the ACTION_MOVE.
                    //过滤无效指针，即不是按下的指针
                    if (!isValidPointerForActionMove(pointerId)) continue;

                    final float x = ev.getX(i);
                    final float y = ev.getY(i);
                    final float dx = x - mInitialMotionX[pointerId];
                    final float dy = y - mInitialMotionY[pointerId];

                    final View toCapture = findTopChildUnder((int) x, (int) y);
                    
                    //检查触摸的有效范围
                    final boolean pastSlop = toCapture != null && checkTouchSlop(toCapture, dx, dy);
                    if (pastSlop) {
                    	//注意：getViewHorizontalDragRange和getViewVerticalDragRange返回大于0,这里仍可以捕获视图，即使DOWN事件不是当前视图
                        // check the callback's
                        // getView[Horizontal|Vertical]DragRange methods to know
                        // if you can move at all along an axis, then see if it
                        // would clamp to the same value. If you can't move at
                        // all in every dimension with a nonzero range, bail.
                        final int oldLeft = toCapture.getLeft();
                        final int targetLeft = oldLeft + (int) dx;
                        final int newLeft = mCallback.clampViewPositionHorizontal(toCapture,
                                targetLeft, (int) dx);
                        final int oldTop = toCapture.getTop();
                        final int targetTop = oldTop + (int) dy;
                        final int newTop = mCallback.clampViewPositionVertical(toCapture, targetTop,
                                (int) dy);
                        final int horizontalDragRange = mCallback.getViewHorizontalDragRange(
                                toCapture);
                        final int verticalDragRange = mCallback.getViewVerticalDragRange(toCapture);
                        if ((horizontalDragRange == 0 || horizontalDragRange > 0
                                && newLeft == oldLeft) && (verticalDragRange == 0
                                || verticalDragRange > 0 && newTop == oldTop)) {
                            break;
                        }
                    }
                    //报告边界触摸开始
                    reportNewEdgeDrags(dx, dy, pointerId);
                    if (mDragState == STATE_DRAGGING) {
                        // Callback might have started an edge drag
                        break;
                    }

                    if (pastSlop && tryCaptureViewForDrag(toCapture, pointerId)) {
                        break;
                    }
                }
                //保存最后触摸信息
                saveLastMotion(ev);
                break;
            }

            case MotionEventCompat.ACTION_POINTER_UP: {
            	//多指情况下，有手指离开的情况
                final int pointerId = ev.getPointerId(actionIndex);
                //清理该指针保存的信息
                clearMotionHistory(pointerId);
                break;
            }

            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL: {
            	//手指离开/触摸取消，取消处理
                cancel();
                break;
            }
        }
	
	//根据是否拖拽状态来拦截事件
        return mDragState == STATE_DRAGGING;
    }
    
public void processTouchEvent(MotionEvent ev) {
        final int action = MotionEventCompat.getActionMasked(ev);
        final int actionIndex = MotionEventCompat.getActionIndex(ev);

        if (action == MotionEvent.ACTION_DOWN) {
            // Reset things for a new event stream, just in case we didn't get
            // the whole previous stream.
            cancel();
        }

        if (mVelocityTracker == null) {
            mVelocityTracker = VelocityTracker.obtain();
        }
        mVelocityTracker.addMovement(ev);

        switch (action) {
            case MotionEvent.ACTION_DOWN: {
                final float x = ev.getX();
                final float y = ev.getY();
                final int pointerId = ev.getPointerId(0);
                final View toCapture = findTopChildUnder((int) x, (int) y);

                saveInitialMotion(x, y, pointerId);

                // Since the parent is already directly processing this touch event,
                // there is no reason to delay for a slop before dragging.
                // Start immediately if possible.
                tryCaptureViewForDrag(toCapture, pointerId);

                final int edgesTouched = mInitialEdgesTouched[pointerId];
                if ((edgesTouched & mTrackingEdges) != 0) {
                    mCallback.onEdgeTouched(edgesTouched & mTrackingEdges, pointerId);
                }
                break;
            }

            case MotionEventCompat.ACTION_POINTER_DOWN: {
                final int pointerId = ev.getPointerId(actionIndex);
                final float x = ev.getX(actionIndex);
                final float y = ev.getY(actionIndex);

                saveInitialMotion(x, y, pointerId);

                // A ViewDragHelper can only manipulate one view at a time.
                if (mDragState == STATE_IDLE) {
                    // If we're idle we can do anything! Treat it like a normal down event.

                    final View toCapture = findTopChildUnder((int) x, (int) y);
                    tryCaptureViewForDrag(toCapture, pointerId);

                    final int edgesTouched = mInitialEdgesTouched[pointerId];
                    if ((edgesTouched & mTrackingEdges) != 0) {
                        mCallback.onEdgeTouched(edgesTouched & mTrackingEdges, pointerId);
                    }
                } else if (isCapturedViewUnder((int) x, (int) y)) {
                    // We're still tracking a captured view. If the same view is under this
                    // point, we'll swap to controlling it with this pointer instead.
                    // (This will still work if we're "catching" a settling view.)

                    tryCaptureViewForDrag(mCapturedView, pointerId);
                }
                break;
            }

            case MotionEvent.ACTION_MOVE: {
                if (mDragState == STATE_DRAGGING) {
                    // If pointer is invalid then skip the ACTION_MOVE.
                    if (!isValidPointerForActionMove(mActivePointerId)) break;

                    final int index = ev.findPointerIndex(mActivePointerId);
                    final float x = ev.getX(index);
                    final float y = ev.getY(index);
                    final int idx = (int) (x - mLastMotionX[mActivePointerId]);
                    final int idy = (int) (y - mLastMotionY[mActivePointerId]);
			
		    //拖拽视图
                    dragTo(mCapturedView.getLeft() + idx, mCapturedView.getTop() + idy, idx, idy);

                    saveLastMotion(ev);
                } else {
                    // Check to see if any pointer is now over a draggable view.
                    //检查是否有任何指针现在位于可拖拽视图上
                    final int pointerCount = ev.getPointerCount();
                    for (int i = 0; i < pointerCount; i++) {
                        final int pointerId = ev.getPointerId(i);

                        // If pointer is invalid then skip the ACTION_MOVE.
                        if (!isValidPointerForActionMove(pointerId)) continue;

                        final float x = ev.getX(i);
                        final float y = ev.getY(i);
                        final float dx = x - mInitialMotionX[pointerId];
                        final float dy = y - mInitialMotionY[pointerId];
			//报告边界触摸拖动
                        reportNewEdgeDrags(dx, dy, pointerId);
                        if (mDragState == STATE_DRAGGING) {
                            // Callback might have started an edge drag.
                            break;
                        }

                        final View toCapture = findTopChildUnder((int) x, (int) y);
                        //检查触摸有效区域，捕获当前触摸视图
                        if (checkTouchSlop(toCapture, dx, dy)
                                && tryCaptureViewForDrag(toCapture, pointerId)) {
                            break;
                        }
                    }
                    saveLastMotion(ev);
                }
                break;
            }

            case MotionEventCompat.ACTION_POINTER_UP: {
                final int pointerId = ev.getPointerId(actionIndex);
                if (mDragState == STATE_DRAGGING && pointerId == mActivePointerId) {
                    // Try to find another pointer that's still holding on to the captured view.
                    //尝试寻找捕获当前视图的其他指针
                    int newActivePointer = INVALID_POINTER;
                    final int pointerCount = ev.getPointerCount();
                    for (int i = 0; i < pointerCount; i++) {
                        final int id = ev.getPointerId(i);
                        if (id == mActivePointerId) {
                            // This one's going away, skip.
                            continue;
                        }

                        final float x = ev.getX(i);
                        final float y = ev.getY(i);
                        if (findTopChildUnder((int) x, (int) y) == mCapturedView
                                && tryCaptureViewForDrag(mCapturedView, id)) {
                            newActivePointer = mActivePointerId;
                            break;
                        }
                    }

                    if (newActivePointer == INVALID_POINTER) {
                        // We didn't find another pointer still touching the view, release it.
                        //没有找到其他指针，释放
                        releaseViewForPointerUp();
                    }
                }
                //清理当前离开指针对应的信息
                clearMotionHistory(pointerId);
                break;
            }

            case MotionEvent.ACTION_UP: {
                if (mDragState == STATE_DRAGGING) {
                    releaseViewForPointerUp();
                }
                cancel();
                break;
            }

            case MotionEvent.ACTION_CANCEL: {
                if (mDragState == STATE_DRAGGING) {
                    dispatchViewReleased(0, 0);
                }
                cancel();
                break;
            }
        }
    }
    
private void saveInitialMotion(float x, float y, int pointerId) {
	//确保初始化变量
        ensureMotionHistorySizeForId(pointerId);
        //保存按下坐标信息
        mInitialMotionX[pointerId] = mLastMotionX[pointerId] = x;
        mInitialMotionY[pointerId] = mLastMotionY[pointerId] = y;
        //保存触摸边界信息s
        mInitialEdgesTouched[pointerId] = getEdgesTouched((int) x, (int) y);
        //保存按下指针信息
        mPointersDown |= 1 << pointerId;
    }
    
private int getEdgesTouched(int x, int y) {
        int result = 0;

        if (x < mParentView.getLeft() + mEdgeSize) result |= EDGE_LEFT;
        if (y < mParentView.getTop() + mEdgeSize) result |= EDGE_TOP;
        if (x > mParentView.getRight() - mEdgeSize) result |= EDGE_RIGHT;
        if (y > mParentView.getBottom() - mEdgeSize) result |= EDGE_BOTTOM;

        return result;
    }
    
    
private boolean checkTouchSlop(View child, float dx, float dy) {
        if (child == null) {
            return false;
        }
        //检查视图可拖动范围
        final boolean checkHorizontal = mCallback.getViewHorizontalDragRange(child) > 0;
        final boolean checkVertical = mCallback.getViewVerticalDragRange(child) > 0;

        if (checkHorizontal && checkVertical) {
            return dx * dx + dy * dy > mTouchSlop * mTouchSlop;
        } else if (checkHorizontal) {
            return Math.abs(dx) > mTouchSlop;
        } else if (checkVertical) {
            return Math.abs(dy) > mTouchSlop;
        }
        return false;
    }
    
//回调方法 
public abstract static class Callback {
        /**
         * Called when the drag state changes. See the <code>STATE_*</code> constants
         * for more information.
         *
         * @param state The new drag state
         *
         * @see #STATE_IDLE
         * @see #STATE_DRAGGING
         * @see #STATE_SETTLING
         */
         //拖动状态改变的回调
        public void onViewDragStateChanged(int state) {}

        /**
         * Called when the captured view's position changes as the result of a drag or settle.
         *
         * @param changedView View whose position changed
         * @param left New X coordinate of the left edge of the view
         * @param top New Y coordinate of the top edge of the view
         * @param dx Change in X position from the last call
         * @param dy Change in Y position from the last call
         */
         //视图位置改变的回调
        public void onViewPositionChanged(View changedView, int left, int top, int dx, int dy) {}

        /**
         * Called when a child view is captured for dragging or settling. The ID of the pointer
         * currently dragging the captured view is supplied. If activePointerId is
         * identified as {@link #INVALID_POINTER} the capture is programmatic instead of
         * pointer-initiated.
         *
         * @param capturedChild Child view that was captured
         * @param activePointerId Pointer id tracking the child capture
         */
         //视图被捕获的回调
        public void onViewCaptured(View capturedChild, int activePointerId) {}

        /**
         * Called when the child view is no longer being actively dragged.
         * The fling velocity is also supplied, if relevant. The velocity values may
         * be clamped to system minimums or maximums.
         *
         * <p>Calling code may decide to fling or otherwise release the view to let it
         * settle into place. It should do so using {@link #settleCapturedViewAt(int, int)}
         * or {@link #flingCapturedView(int, int, int, int)}. If the Callback invokes
         * one of these methods, the ViewDragHelper will enter {@link #STATE_SETTLING}
         * and the view capture will not fully end until it comes to a complete stop.
         * If neither of these methods is invoked before <code>onViewReleased</code> returns,
         * the view will stop in place and the ViewDragHelper will return to
         * {@link #STATE_IDLE}.</p>
         *
         * @param releasedChild The captured child view now being released
         * @param xvel X velocity of the pointer as it left the screen in pixels per second.
         * @param yvel Y velocity of the pointer as it left the screen in pixels per second.
         */
         //视图从被拖动到被释放的回调
        public void onViewReleased(View releasedChild, float xvel, float yvel) {}

        /**
         * Called when one of the subscribed edges in the parent view has been touched
         * by the user while no child view is currently captured.
         *
         * @param edgeFlags A combination of edge flags describing the edge(s) currently touched
         * @param pointerId ID of the pointer touching the described edge(s)
         * @see #EDGE_LEFT
         * @see #EDGE_TOP
         * @see #EDGE_RIGHT
         * @see #EDGE_BOTTOM
         */
         //触摸边界回调
        public void onEdgeTouched(int edgeFlags, int pointerId) {}

        /**
         * Called when the given edge may become locked. This can happen if an edge drag
         * was preliminarily rejected before beginning, but after {@link #onEdgeTouched(int, int)}
         * was called. This method should return true to lock this edge or false to leave it
         * unlocked. The default behavior is to leave edges unlocked.
         *
         * @param edgeFlags A combination of edge flags describing the edge(s) locked
         * @return true to lock the edge, false to leave it unlocked
         */
        public boolean onEdgeLock(int edgeFlags) {
            return false;
        }

        /**
         * Called when the user has started a deliberate drag away from one
         * of the subscribed edges in the parent view while no child view is currently captured.
         *
         * @param edgeFlags A combination of edge flags describing the edge(s) dragged
         * @param pointerId ID of the pointer touching the described edge(s)
         * @see #EDGE_LEFT
         * @see #EDGE_TOP
         * @see #EDGE_RIGHT
         * @see #EDGE_BOTTOM
         */
         //边界开始触摸滚动
        public void onEdgeDragStarted(int edgeFlags, int pointerId) {}

        /**
         * Called to determine the Z-order of child views.
         *
         * @param index the ordered position to query for
         * @return index of the view that should be ordered at position <code>index</code>
         */
         //获取子视图的顺序
        public int getOrderedChildIndex(int index) {
            return index;
        }
	
	
	//注意:这两个方法的作用有:
	//1.当DOWN事件被子视图拦截了，在shouldInterceptTouchEvent中会调用checkTouchSlop，从而调用这两个方法，来确定是否再次捕获当前子视图，拦截事件
	//2.调用settleCapturedViewAt时，来确定Settle(惯性滚动)的持续时间
        /**
         * Return the magnitude of a draggable child view's horizontal range of motion in pixels.
         * This method should return 0 for views that cannot move horizontally.
         *
         * @param child Child view to check
         * @return range of horizontal motion in pixels
         */
         //获取该视图水平滚动的范围
        public int getViewHorizontalDragRange(View child) {
            return 0;
        }

        /**
         * Return the magnitude of a draggable child view's vertical range of motion in pixels.
         * This method should return 0 for views that cannot move vertically.
         *
         * @param child Child view to check
         * @return range of vertical motion in pixels
         */
         //获取该视图垂直滚动的范围
        public int getViewVerticalDragRange(View child) {
            return 0;
        }
        
        

        /**
         * Called when the user's input indicates that they want to capture the given child view
         * with the pointer indicated by pointerId. The callback should return true if the user
         * is permitted to drag the given view with the indicated pointer.
         *
         * <p>ViewDragHelper may call this method multiple times for the same view even if
         * the view is already captured; this indicates that a new pointer is trying to take
         * control of the view.</p>
         *
         * <p>If this method returns true, a call to {@link #onViewCaptured(android.view.View, int)}
         * will follow if the capture is successful.</p>
         *
         * @param child Child the user is attempting to capture
         * @param pointerId ID of the pointer attempting the capture
         * @return true if capture should be allowed, false otherwise
         */
         //该视图能否被捕获
        public abstract boolean tryCaptureView(View child, int pointerId);

        /**
         * Restrict the motion of the dragged child view along the horizontal axis.
         * The default implementation does not allow horizontal motion; the extending
         * class must override this method and provide the desired clamping.
         *
         *
         * @param child Child view being dragged
         * @param left Attempted motion along the X axis
         * @param dx Proposed change in position for left
         * @return The new clamped position for left
         */
         //该视图水平拖动距离
        public int clampViewPositionHorizontal(View child, int left, int dx) {
            return 0;
        }

        /**
         * Restrict the motion of the dragged child view along the vertical axis.
         * The default implementation does not allow vertical motion; the extending
         * class must override this method and provide the desired clamping.
         *
         *
         * @param child Child view being dragged
         * @param top Attempted motion along the Y axis
         * @param dy Proposed change in position for top
         * @return The new clamped position for top
         */
          //该视图垂直拖动距离
        public int clampViewPositionVertical(View child, int top, int dy) {
            return 0;
        }
    }

```

