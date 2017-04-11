```java
    public void scrollTo(int x, int y) {
        if (mScrollX != x || mScrollY != y) {
           //重新赋新值
            int oldX = mScrollX;
            int oldY = mScrollY;
            mScrollX = x;
            mScrollY = y;
            //清除父视图应该清除当前View的缓存,不invalidate
            invalidateParentCaches();
            //滚动变化处理 
            onScrollChanged(mScrollX, mScrollY, oldX, oldY);
            //唤醒滚动条
            if (!awakenScrollBars()) {
            	//如果SrollCache == null 或者 不需要绘制滚动条，则直接调用invalidate()
                postInvalidateOnAnimation();
            }
        }
    }
    
    protected boolean awakenScrollBars(int startDelay, boolean invalidate) {
        final ScrollabilityCache scrollCache = mScrollCache;

        if (scrollCache == null || !scrollCache.fadeScrollBars) {
            return false;
        }

	//滚动条样式
        if (scrollCache.scrollBar == null) {
            scrollCache.scrollBar = new ScrollBarDrawable();
            scrollCache.scrollBar.setState(getDrawableState());
            scrollCache.scrollBar.setCallback(this);
        }

	//是否启用水平滚动条或者垂直滚动条绘制
        if (isHorizontalScrollBarEnabled() || isVerticalScrollBarEnabled()) {
	
	    
            if (invalidate) {
                // Invalidate to show the scrollbars
                //调用invalidate()来绘制滚动条
                postInvalidateOnAnimation();
            }

            if (scrollCache.state == ScrollabilityCache.OFF) {
            	//当前滚动条不可见，则增加淡出滚动条的延迟时间
                // FIXME: this is copied from WindowManagerService.
                // We should get this value from the system when it
                // is possible to do so.
                final int KEY_REPEAT_FIRST_DELAY = 750;
                startDelay = Math.max(KEY_REPEAT_FIRST_DELAY, startDelay);
            }

            // Tell mScrollCache when we should start fading. This may
            // extend the fade start time if one was already scheduled
            long fadeStartTime = AnimationUtils.currentAnimationTimeMillis() + startDelay;
            scrollCache.fadeStartTime = fadeStartTime;
            scrollCache.state = ScrollabilityCache.ON;

            // Schedule our fader to run, unscheduling any old ones first
            if (mAttachInfo != null) {
                mAttachInfo.mHandler.removeCallbacks(scrollCache);
                mAttachInfo.mHandler.postAtTime(scrollCache, fadeStartTime);
            }

            return true;
        }

        return false;
    }
    
     protected boolean awakenScrollBars() {
     //ScrollabilityCache保存滚动时视图使用的字段
        return mScrollCache != null &&
                awakenScrollBars(mScrollCache.scrollBarDefaultDelayBeforeFade, true);
    }
    
    protected void onScrollChanged(int l, int t, int oldl, int oldt) {
    
    	//通知此视图的辅助功能(Accessibility)状态已更改
    	//accessibility state改变原因有，例如视图大小更改
        notifySubtreeAccessibilityStateChangedIfNeeded();

	//设置回调事件，发送滚动视图辅助功能(Accessibility)事件
        if (AccessibilityManager.getInstance(mContext).isEnabled()) {
            postSendViewScrolledAccessibilityEventCallback();
        }

	//标识背景大小改变,重新绘制背景
        mBackgroundSizeChanged = true;
        if (mForegroundInfo != null) {
        //存在前景，标识前景大小改变,重新绘制前景
            mForegroundInfo.mBoundsChanged = true;
        }

	//标识视图滚动改变
        final AttachInfo ai = mAttachInfo;
        if (ai != null) {
            ai.mViewScrollChanged = true;
        }
	
	//监听事件回调
        if (mListenerInfo != null && mListenerInfo.mOnScrollChangeListener != null) {
            mListenerInfo.mOnScrollChangeListener.onScrollChange(this, l, t, oldl, oldt);
        }
    }
    
     boolean draw(Canvas canvas, ViewGroup parent, long drawingTime){
     	......
     	int sx = 0;
        int sy = 0;
        if (!drawingWithRenderNode) {
            computeScroll();
            sx = mScrollX;
            sy = mScrollY;
        }

        final boolean drawingWithDrawingCache = cache != null && !drawingWithRenderNode;
        final boolean offsetForScroll = cache == null && !drawingWithRenderNode;

        int restoreTo = -1;
        if (!drawingWithRenderNode || transformToApply != null) {
            restoreTo = canvas.save();
        }
        if (offsetForScroll) {
        //通过位移画布,来进行滚动效果
        //mLeft-sx,mTop-sy，所以当scrollx为正时候，向左滚动。srollY为正时候，向上滚动
            canvas.translate(mLeft - sx, mTop - sy);
        } else {
            if (!drawingWithRenderNode) {
                canvas.translate(mLeft, mTop);
            }
            if (scalingRequired) {
                if (drawingWithRenderNode) {
                    // TODO: Might not need this if we put everything inside the DL
                    restoreTo = canvas.save();
                }
                // mAttachInfo cannot be null, otherwise scalingRequired == false
                final float scale = 1.0f / mAttachInfo.mApplicationScale;
                canvas.scale(scale, scale);
            }
        }
     	......
     }
    
    
    

```

