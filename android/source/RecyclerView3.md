## Touch

### onInterceptTouchEvent

``` java
@Override                                                                                     
public boolean onInterceptTouchEvent(MotionEvent e) {                                         
    if (mLayoutFrozen) {                                                                      
        // When layout is frozen,  RV does not intercept the motion event.                    
        // A child view e.g. a button may still get the click.                                
        return false;                                                                         
    }                                                                                         
    if (dispatchOnItemTouchIntercept(e)) {
      	//存在 OnItemTouchListener 消费
        cancelTouch();                                                                        
        return true;                                                                          
    }                                                                                         
                                                                                              
    if (mLayout == null) {                                                                    
        return false;                                                                         
    }                                                                                         
                                                                                              
    final boolean canScrollHorizontally = mLayout.canScrollHorizontally();                    
    final boolean canScrollVertically = mLayout.canScrollVertically();                        
                                                                                              
    if (mVelocityTracker == null) {                                                           
        mVelocityTracker = VelocityTracker.obtain();                                          
    }                                                                                         
    mVelocityTracker.addMovement(e);                                                          
                                                                                              
    final int action = e.getActionMasked();                                                   
    final int actionIndex = e.getActionIndex();                                               
                                                                                              
    switch (action) {                                                                         
        case MotionEvent.ACTION_DOWN:                                                         
            if (mIgnoreMotionEventTillDown) {                                                 
                mIgnoreMotionEventTillDown = false;                                           
            }                                                                                 
            mScrollPointerId = e.getPointerId(0);                                             
            mInitialTouchX = mLastTouchX = (int) (e.getX() + 0.5f);                           
            mInitialTouchY = mLastTouchY = (int) (e.getY() + 0.5f);                           
                                                                                              
            if (mScrollState == SCROLL_STATE_SETTLING) {
              	//当前滚动状态为惯性滚动
              	//禁止父视图拦截事件
                getParent().requestDisallowInterceptTouchEvent(true);
              	//设置滚动状态为拖动滚动
                setScrollState(SCROLL_STATE_DRAGGING);                                        
            }                                                                                 
                                                                                              
            // Clear the nested offsets                                                       
            mNestedOffsets[0] = mNestedOffsets[1] = 0;                                        
            // 嵌套滚动                                                                                  
            int nestedScrollAxis = ViewCompat.SCROLL_AXIS_NONE;                               
            if (canScrollHorizontally) {                                                      
                nestedScrollAxis |= ViewCompat.SCROLL_AXIS_HORIZONTAL;                        
            }                                                                                 
            if (canScrollVertically) {                                                        
                nestedScrollAxis |= ViewCompat.SCROLL_AXIS_VERTICAL;                          
            }                                                                                 
            startNestedScroll(nestedScrollAxis, TYPE_TOUCH);                                  
            break;                                                                            
                                                                                              
        case MotionEvent.ACTION_POINTER_DOWN:
        	//多指处理
            mScrollPointerId = e.getPointerId(actionIndex);                                   
            mInitialTouchX = mLastTouchX = (int) (e.getX(actionIndex) + 0.5f);                
            mInitialTouchY = mLastTouchY = (int) (e.getY(actionIndex) + 0.5f);                
            break;                                                                            
                                                                                              
        case MotionEvent.ACTION_MOVE: {                                                       
            final int index = e.findPointerIndex(mScrollPointerId);                           
            if (index < 0) {                                                                  
                Log.e(TAG, "Error processing scroll; pointer index for id "                   
                        + mScrollPointerId + " not found. Did any MotionEvents get skipped?");
                return false;                                                                 
            }                                                                                 
                                                                                              
            final int x = (int) (e.getX(index) + 0.5f);                                       
            final int y = (int) (e.getY(index) + 0.5f);                                       
            if (mScrollState != SCROLL_STATE_DRAGGING) {                                      
                final int dx = x - mInitialTouchX;                                            
                final int dy = y - mInitialTouchY;                                            
                boolean startScroll = false;                                                  
                if (canScrollHorizontally && Math.abs(dx) > mTouchSlop) {                     
                    mLastTouchX = x;                                                          
                    startScroll = true;                                                       
                }                                                                             
                if (canScrollVertically && Math.abs(dy) > mTouchSlop) {                       
                    mLastTouchY = y;                                                          
                    startScroll = true;                                                       
                }                                                                             
                if (startScroll) {
                  	//设置为拖动滚动
                    setScrollState(SCROLL_STATE_DRAGGING);                                    
                }                                                                             
            }                                                                                 
        } break;                                                                              
                                                                                              
        case MotionEvent.ACTION_POINTER_UP: {
          	//多指处理
            onPointerUp(e);                                                                   
        } break;                                                                              
                                                                                              
        case MotionEvent.ACTION_UP: {
          	//手指抬起处理
            mVelocityTracker.clear();                                                         
            stopNestedScroll(TYPE_TOUCH);                                                     
        } break;                                                                              
                                                                                              
        case MotionEvent.ACTION_CANCEL: {
          	//取消处理
            cancelTouch();                                                                    
        }                                                                                     
    }                                                                                         
    return mScrollState == SCROLL_STATE_DRAGGING;                                             
}                                                                                             
```

``` java
private boolean dispatchOnItemTouchIntercept(MotionEvent e) {                                   
    final int action = e.getAction();                                                           
    if (action == MotionEvent.ACTION_CANCEL || action == MotionEvent.ACTION_DOWN) {             
        mActiveOnItemTouchListener = null;                                                      
    }                                                                                           
                                                                                                
    final int listenerCount = mOnItemTouchListeners.size();                                     
    for (int i = 0; i < listenerCount; i++) {                                                   
        final OnItemTouchListener listener = mOnItemTouchListeners.get(i);                      
        if (listener.onInterceptTouchEvent(this, e) && action != MotionEvent.ACTION_CANCEL) {   
            mActiveOnItemTouchListener = listener;                                              
            return true;                                                                        
        }                                                                                       
    }                                                                                           
    return false;                                                                               
}                                                                                               
```

###  requestDisallowInterceptTouchEvent

``` java
@Override                                                                  
public void requestDisallowInterceptTouchEvent(boolean disallowIntercept) {
    final int listenerCount = mOnItemTouchListeners.size();                
    for (int i = 0; i < listenerCount; i++) {
      	//分发给 OnItemTouchListener 处理
        final OnItemTouchListener listener = mOnItemTouchListeners.get(i); 
        listener.onRequestDisallowInterceptTouchEvent(disallowIntercept);  
    }                                                                      
    super.requestDisallowInterceptTouchEvent(disallowIntercept);           
}                                                                          
```

### onTouchEvent

``` java
@Override                                                                                         
public boolean onTouchEvent(MotionEvent e) {                                                      
    if (mLayoutFrozen || mIgnoreMotionEventTillDown) {                                            
        return false;                                                                             
    }                                                                                             
    if (dispatchOnItemTouch(e)) {
      	//分发给 OnItemTouch
        cancelTouch();                                                                            
        return true;                                                                              
    }                                                                                             
                                                                                                  
    if (mLayout == null) {                                                                        
        return false;                                                                             
    }                                                                                             
                                                                                                  
    final boolean canScrollHorizontally = mLayout.canScrollHorizontally();                        
    final boolean canScrollVertically = mLayout.canScrollVertically();                            
                                                                                                  
    if (mVelocityTracker == null) {                                                               
        mVelocityTracker = VelocityTracker.obtain();                                              
    }                                                                                             
    boolean eventAddedToVelocityTracker = false;                                                  
                                                                                                  
    final MotionEvent vtev = MotionEvent.obtain(e);                                               
    final int action = e.getActionMasked();                                                       
    final int actionIndex = e.getActionIndex();                                                   
                                                                                                  
    if (action == MotionEvent.ACTION_DOWN) {                                                      
        mNestedOffsets[0] = mNestedOffsets[1] = 0;                                                
    }                                                                                             
    vtev.offsetLocation(mNestedOffsets[0], mNestedOffsets[1]);                                    
                                                                                                  
    switch (action) {                                                                             
        case MotionEvent.ACTION_DOWN: {                                                           
          	//嵌套滚动
            mScrollPointerId = e.getPointerId(0);                                                 
            mInitialTouchX = mLastTouchX = (int) (e.getX() + 0.5f);                               
            mInitialTouchY = mLastTouchY = (int) (e.getY() + 0.5f);                               
                                                                                                  
            int nestedScrollAxis = ViewCompat.SCROLL_AXIS_NONE;                                   
            if (canScrollHorizontally) {                                                          
                nestedScrollAxis |= ViewCompat.SCROLL_AXIS_HORIZONTAL;                            
            }                                                                                     
            if (canScrollVertically) {                                                            
                nestedScrollAxis |= ViewCompat.SCROLL_AXIS_VERTICAL;                              
            }                                                                                     
            startNestedScroll(nestedScrollAxis, TYPE_TOUCH);                                      
        } break;                                                                                  
                                                                                                  
        case MotionEvent.ACTION_POINTER_DOWN: {                                                   
          	//手指处理
            mScrollPointerId = e.getPointerId(actionIndex);                                       
            mInitialTouchX = mLastTouchX = (int) (e.getX(actionIndex) + 0.5f);                    
            mInitialTouchY = mLastTouchY = (int) (e.getY(actionIndex) + 0.5f);                    
        } break;                                                                                  
                                                                                                  
        case MotionEvent.ACTION_MOVE: {                                                           
            final int index = e.findPointerIndex(mScrollPointerId);                               
            if (index < 0) {                                                                      
                Log.e(TAG, "Error processing scroll; pointer index for id "                       
                        + mScrollPointerId + " not found. Did any MotionEvents get skipped?");    
                return false;                                                                     
            }                                                                                     
                                                                                                  
            final int x = (int) (e.getX(index) + 0.5f);                                           
            final int y = (int) (e.getY(index) + 0.5f);                                           
            int dx = mLastTouchX - x;                                                             
            int dy = mLastTouchY - y;                                                             
                                                                                                  
            if (dispatchNestedPreScroll(dx, dy, mScrollConsumed, mScrollOffset, TYPE_TOUCH)) {    
                dx -= mScrollConsumed[0];                                                         
                dy -= mScrollConsumed[1];                                                         
                vtev.offsetLocation(mScrollOffset[0], mScrollOffset[1]);                          
                // Updated the nested offsets                                                     
                mNestedOffsets[0] += mScrollOffset[0];                                            
                mNestedOffsets[1] += mScrollOffset[1];                                            
            }                                                                                     
                                                                                                  
            if (mScrollState != SCROLL_STATE_DRAGGING) {                                          
                boolean startScroll = false;                                                      
                if (canScrollHorizontally && Math.abs(dx) > mTouchSlop) {                         
                    if (dx > 0) {                                                                 
                        dx -= mTouchSlop;                                                         
                    } else {                                                                      
                        dx += mTouchSlop;                                                         
                    }                                                                             
                    startScroll = true;                                                           
                }                                                                                 
                if (canScrollVertically && Math.abs(dy) > mTouchSlop) {                           
                    if (dy > 0) {                                                                 
                        dy -= mTouchSlop;                                                         
                    } else {                                                                      
                        dy += mTouchSlop;                                                         
                    }                                                                             
                    startScroll = true;                                                           
                }                                                                                 
                if (startScroll) {                                                                
                  	//设置为拖动滚动
                    setScrollState(SCROLL_STATE_DRAGGING);                                        
                }                                                                                 
            }                                                                                     
                                                                                                  
            if (mScrollState == SCROLL_STATE_DRAGGING) {                                          
                mLastTouchX = x - mScrollOffset[0];                                               
                mLastTouchY = y - mScrollOffset[1];                                               
				//开始滚动
                if (scrollByInternal(                                                             
                        canScrollHorizontally ? dx : 0,                                           
                        canScrollVertically ? dy : 0,                                             
                        vtev)) {                                                                  
                    getParent().requestDisallowInterceptTouchEvent(true);                         
                }                                                                                 
                if (mGapWorker != null && (dx != 0 || dy != 0)) {                                 
                    mGapWorker.postFromTraversal(this, dx, dy);                                   
                }                                                                                 
            }                                                                                     
        } break;                                                                                  
                                                                                                  
        case MotionEvent.ACTION_POINTER_UP: {                                                     
          	//多指处理
            onPointerUp(e);                                                                       
        } break;                                                                                  
                                                                                                  
        case MotionEvent.ACTION_UP: {                                                             
            mVelocityTracker.addMovement(vtev);                                                   
            eventAddedToVelocityTracker = true;                                                   
            mVelocityTracker.computeCurrentVelocity(1000, mMaxFlingVelocity);                     
            final float xvel = canScrollHorizontally                                              
                    ? -mVelocityTracker.getXVelocity(mScrollPointerId) : 0;                       
            final float yvel = canScrollVertically                                                
                    ? -mVelocityTracker.getYVelocity(mScrollPointerId) : 0;                       
          	// fling
            if (!((xvel != 0 || yvel != 0) && fling((int) xvel, (int) yvel))) {                   
              	// 不处理 fling，设置滚动状态为滚动静止
                setScrollState(SCROLL_STATE_IDLE);                                                
            }                                                                                     
            resetTouch();                                                                         
        } break;                                                                                  
                                                                                                  
        case MotionEvent.ACTION_CANCEL: {                                                         
            cancelTouch();                                                                        
        } break;                                                                                  
    }                                                                                             
                                                                                                  
    if (!eventAddedToVelocityTracker) {                                                           
        mVelocityTracker.addMovement(vtev);                                                       
    }                                                                                             
    vtev.recycle();                                                                               
                                                                                                  
    return true;                                                                                  
}                                                                                                 
```

``` java
/**                                                                                         
 * Does not perform bounds checking. Used by internal methods that have already validated in
 * <p>                                                                                      
 * It also reports any unused scroll request to the related EdgeEffect.                     
 *                                                                                          
 * @param x The amount of horizontal scroll request                                         
 * @param y The amount of vertical scroll request                                           
 * @param ev The originating MotionEvent, or null if not from a touch event.                
 *                                                                                          
 * @return Whether any scroll was consumed in either direction.                             
 */                                                                                         
boolean scrollByInternal(int x, int y, MotionEvent ev) {                                    
    int unconsumedX = 0, unconsumedY = 0;                                                   
    int consumedX = 0, consumedY = 0;                                                       
    // 消费等待的更新操作                                                                                        
    consumePendingUpdateOperations();                                                       
    if (mAdapter != null) {                                                                 
        eatRequestLayout();                                                                 
        onEnterLayoutOrScroll();                                                            
        TraceCompat.beginSection(TRACE_SCROLL_TAG);                                         
        fillRemainingScrollValues(mState);                                                  
        if (x != 0) {
          	// LayoutManager 进行消费滚动
            consumedX = mLayout.scrollHorizontallyBy(x, mRecycler, mState);                 
            unconsumedX = x - consumedX;                                                    
        }                                                                                   
        if (y != 0) {
          	// LayoutManager 进行消费滚动
            consumedY = mLayout.scrollVerticallyBy(y, mRecycler, mState);                   
            unconsumedY = y - consumedY;                                                    
        }                                                                                   
        TraceCompat.endSection();                                                           
        repositionShadowingViews();                                                         
        onExitLayoutOrScroll();                                                             
        resumeRequestLayout(false);                                                         
    }                                                                                       
    if (!mItemDecorations.isEmpty()) {                                                      
        invalidate();                                                                       
    }                                                                                       
    
  	// 分发嵌套滚动
    if (dispatchNestedScroll(consumedX, consumedY, unconsumedX, unconsumedY, mScrollOffset, 
            TYPE_TOUCH)) {                                                                  
        // Update the last touch co-ords, taking any scroll offset into account             
        mLastTouchX -= mScrollOffset[0];                                                    
        mLastTouchY -= mScrollOffset[1];                                                    
        if (ev != null) {                                                                   
            ev.offsetLocation(mScrollOffset[0], mScrollOffset[1]);                          
        }                                                                                   
        mNestedOffsets[0] += mScrollOffset[0];                                              
        mNestedOffsets[1] += mScrollOffset[1];                                              
    } else if (getOverScrollMode() != View.OVER_SCROLL_NEVER) {
      	//滚动阴影效果
        if (ev != null && !MotionEventCompat.isFromSource(ev, InputDevice.SOURCE_MOUSE)) {  
            pullGlows(ev.getX(), unconsumedX, ev.getY(), unconsumedY);                      
        }                                                                                   
        considerReleasingGlowsOnScroll(x, y);                                               
    }                                                                                       
    if (consumedX != 0 || consumedY != 0) {
      	//分发滚动通知事件
        dispatchOnScrolled(consumedX, consumedY);                                           
    }                                                                                       
    if (!awakenScrollBars()) {                                                              
        invalidate();                                                                       
    }                                                                                       
    return consumedX != 0 || consumedY != 0;                                                
}                                                                                           
```



### fling

``` java
/**                                                                                           
 * Begin a standard fling with an initial velocity along each axis in pixels per second.      
 * If the velocity given is below the system-defined minimum this method will return false    
 * and no fling will occur.                                                                   
 *                                                                                            
 * @param velocityX Initial horizontal velocity in pixels per second                          
 * @param velocityY Initial vertical velocity in pixels per second                            
 * @return true if the fling was started, false if the velocity was too low to fling or       
 * LayoutManager does not support scrolling in the axis fling is issued.                      
 *                                                                                            
 * @see LayoutManager#canScrollVertically()                                                   
 * @see LayoutManager#canScrollHorizontally()                                                 
 */                                                                                           
public boolean fling(int velocityX, int velocityY) {                                          
    if (mLayout == null) {                                                                    
        Log.e(TAG, "Cannot fling without a LayoutManager set. "                               
                + "Call setLayoutManager with a non-null argument.");                         
        return false;                                                                         
    }                                                                                         
    if (mLayoutFrozen) {                                                                      
        return false;                                                                         
    }                                                                                         
                                                                                              
    final boolean canScrollHorizontal = mLayout.canScrollHorizontally();                      
    final boolean canScrollVertical = mLayout.canScrollVertically();                          
                                                                                              
    if (!canScrollHorizontal || Math.abs(velocityX) < mMinFlingVelocity) {                    
        velocityX = 0;                                                                        
    }                                                                                         
    if (!canScrollVertical || Math.abs(velocityY) < mMinFlingVelocity) {                      
        velocityY = 0;                                                                        
    }                                                                                         
    if (velocityX == 0 && velocityY == 0) {                                                   
        // If we don't have any velocity, return false                                        
        return false;                                                                         
    }                                                                                         
                                                                                              
    if (!dispatchNestedPreFling(velocityX, velocityY)) {
      	// 分发嵌套滚动
        final boolean canScroll = canScrollHorizontal || canScrollVertical;
        dispatchNestedFling(velocityX, velocityY, canScroll);                                 
                                                                                              
        if (mOnFlingListener != null && mOnFlingListener.onFling(velocityX, velocityY)) {
          	//OnFlingListener 消费了
            return true;                                                                      
        }                                                                                     
                                                                                              
        if (canScroll) {                                                                      
            int nestedScrollAxis = ViewCompat.SCROLL_AXIS_NONE;                               
            if (canScrollHorizontal) {                                                        
                nestedScrollAxis |= ViewCompat.SCROLL_AXIS_HORIZONTAL;                        
            }                                                                                 
            if (canScrollVertical) {                                                          
                nestedScrollAxis |= ViewCompat.SCROLL_AXIS_VERTICAL;                          
            }
          	//分发嵌套滚动
            startNestedScroll(nestedScrollAxis, TYPE_NON_TOUCH);                              
                                                                                              
            velocityX = Math.max(-mMaxFlingVelocity, Math.min(velocityX, mMaxFlingVelocity)); 
            velocityY = Math.max(-mMaxFlingVelocity, Math.min(velocityY, mMaxFlingVelocity));
          	//fling
            mViewFlinger.fling(velocityX, velocityY);                                         
            return true;                                                                      
        }                                                                                     
    }                                                                                         
    return false;                                                                             
}                                                                                             
```

### LayoutManager

RecyclerView 的 scroll 和 fling 都会调用到 `LayoutManager.scrollHorizontallyBy()` 或者 `LayoutManager.scrollVerticallyBy()` 进行实际滚动处理。

我们还是以 `LinearLayoutManager.scrollVerticallyBy()` 进行分析：

``` java
int scrollBy(int dy, RecyclerView.Recycler recycler, RecyclerView.State state) {                
    if (getChildCount() == 0 || dy == 0) {                                                      
        return 0;                                                                               
    }                                                                                           
    mLayoutState.mRecycle = true;                                                               
    ensureLayoutState();                                                                        
    final int layoutDirection = dy > 0 ? LayoutState.LAYOUT_END : LayoutState.LAYOUT_START;     
    final int absDy = Math.abs(dy);
  	// 更新 layout state
    updateLayoutState(layoutDirection, absDy, true, state);
  	// 调用 fill 进行填充
    final int consumed = mLayoutState.mScrollingOffset                                          
            + fill(recycler, mLayoutState, state, false);                                       
    if (consumed < 0) {                                                                         
        if (DEBUG) {                                                                            
            Log.d(TAG, "Don't have any more elements to scroll");                               
        }                                                                                       
        return 0;                                                                               
    }                                                                                           
    final int scrolled = absDy > consumed ? layoutDirection * consumed : dy;
  	// 平移 child view，让 fill 的 view 处于可见范围。
    mOrientationHelper.offsetChildren(-scrolled);                                               
    if (DEBUG) {                                                                                
        Log.d(TAG, "scroll req: " + dy + " scrolled: " + scrolled);                             
    }                                                                                           
    mLayoutState.mLastScrollDelta = scrolled;                                                   
    return scrolled;                                                                            
}                                                                                               
```

``` java
// 根据 scroll 更新需要重新 fill 添加的空间距离。
private void updateLayoutState(int layoutDirection, int requiredSpace,                         
        boolean canUseExistingSpace, RecyclerView.State state) {                               
    // If parent provides a hint, don't measure unlimited.                                     
    mLayoutState.mInfinite = resolveIsInfinite();
  	// 如果存在 target position 返回总空间
    mLayoutState.mExtra = getExtraLayoutSpace(state);                                          
    mLayoutState.mLayoutDirection = layoutDirection;                                           
    int scrollingOffset;                                                                       
    if (layoutDirection == LayoutState.LAYOUT_END) {
      	// 从 End 开始布局
        mLayoutState.mExtra += mOrientationHelper.getEndPadding();                             
        // 获取当前布局方向的第一个 child view。                                   
        final View child = getChildClosestToEnd();                                             
        // the direction in which we are traversing children                                   
        mLayoutState.mItemDirection = mShouldReverseLayout ? LayoutState.ITEM_DIRECTION_HEAD   
                : LayoutState.ITEM_DIRECTION_TAIL;                                             
        mLayoutState.mCurrentPosition = getPosition(child) + mLayoutState.mItemDirection;      
        mLayoutState.mOffset = mOrientationHelper.getDecoratedEnd(child);                      
        // 计算我们在不添加新的 child view 的情况下，最大能滚动多少距离（独立于 layout）
        scrollingOffset = mOrientationHelper.getDecoratedEnd(child)                            
                - mOrientationHelper.getEndAfterPadding();                                     
                                                                                               
    } else {                                                                                   
        final View child = getChildClosestToStart();                                           
        mLayoutState.mExtra += mOrientationHelper.getStartAfterPadding();                      
        mLayoutState.mItemDirection = mShouldReverseLayout ? LayoutState.ITEM_DIRECTION_TAIL   
                : LayoutState.ITEM_DIRECTION_HEAD;                                             
        mLayoutState.mCurrentPosition = getPosition(child) + mLayoutState.mItemDirection;      
        mLayoutState.mOffset = mOrientationHelper.getDecoratedStart(child);                    
        scrollingOffset = -mOrientationHelper.getDecoratedStart(child)                         
                + mOrientationHelper.getStartAfterPadding();                                   
    }                                                                                          
    mLayoutState.mAvailable = requiredSpace;                                                   
    if (canUseExistingSpace) {                                                                 
        mLayoutState.mAvailable -= scrollingOffset;                                            
    }
  	// 保存滚动多少距离，而不用创建一个新的 view。这是高效回收所需的设置。
    mLayoutState.mScrollingOffset = scrollingOffset;                                           
}                                                                                              
```





​                                                                                         

