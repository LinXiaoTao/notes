## 构造函数

AdapterHelper 队列化并且处理 Adapter 更新操作。

``` java
void initAdapterManager(){
  
}
```



ChildHelper 帮助管理 RecyclerView child view。

``` java
private void initChildrenHelper(){
  
}
```



ItemAnimatorListener 是 ItemAnimator 的事件监听接口。

``` java
private ItemAnimator.ItemAnimatorListener mItemAnimatorListener =
            new ItemAnimatorRestoreListener();
mItemAnimator.setListener(mItemAnimatorListener);
```



## Measure, Layout and Draw

### Measure

``` java
@Override                                                                                       
protected void onMeasure(int widthSpec, int heightSpec) {                                       
    if (mLayout == null) {                                                                      
      	//LayoutManager is null
        defaultOnMeasure(widthSpec, heightSpec);                                                
        return;                                                                                 
    }                                                                                           
    if (mLayout.mAutoMeasure) {
      	//auto measure
        final int widthMode = MeasureSpec.getMode(widthSpec);                                   
        final int heightMode = MeasureSpec.getMode(heightSpec);                                 
        final boolean skipMeasure = widthMode == MeasureSpec.EXACTLY                            
                && heightMode == MeasureSpec.EXACTLY;                                           
        mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);                            
        if (skipMeasure || mAdapter == null) {                                                  
          	//测量参数为固定值或者 Adapter is null
            return;                                                                             
        }                                                                                       
        if (mState.mLayoutStep == State.STEP_START) {
          	//第一步，pre-layout
            dispatchLayoutStep1();                                                              
        }                                                                                       
      	// 在第二步时设置尺寸，pre-layout 应该与旧尺寸一致。
        mLayout.setMeasureSpecs(widthSpec, heightSpec);                                         
        mState.mIsMeasuring = true;
      	//第二步，actual-layout
        dispatchLayoutStep2();                                                                  
                                                                                                
        // now we can get the width and height from the children.                               
        mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);                        
                                                                                                
      	//如果 RecyclerView 不是使用精确的宽和高，并且如果至少有一个 child view，不是使用精确的宽和高，我们应该重新测量。
        if (mLayout.shouldMeasureTwice()) {                                                     
          	//应该测量两次
            mLayout.setMeasureSpecs(                                                            
                    MeasureSpec.makeMeasureSpec(getMeasuredWidth(), MeasureSpec.EXACTLY),       
                    MeasureSpec.makeMeasureSpec(getMeasuredHeight(), MeasureSpec.EXACTLY));     
            mState.mIsMeasuring = true;                                                         
          	//重新进行第二步，actual-layout
            dispatchLayoutStep2();                                                              
            // now we can get the width and height from the children.                           
            mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);                    
        }                                                                                       
    } else {
      	//not auto measure
        if (mHasFixedSize) {                                                                    
          	//固定不变的尺寸，调用 LayoutMananger.onMeasure 测量
            mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);                        
            return;                                                                             
        }                                                                                       
        // 自定义测量                                                                     
        if (mAdapterUpdateDuringMeasure) {                                                      
          	//如果在 measure 过程中出现 Adapter 更新
            eatRequestLayout();                                                                 
            onEnterLayoutOrScroll();                                                            
          	//根据 Adapter 的更新来计算我们想要运行的动画类型，在 dispatchLayoutStep1 调用过一次。
            processAdapterUpdatesAndSetAnimationFlags();                                        
            onExitLayoutOrScroll();                                                             
                                                                                                
            if (mState.mRunPredictiveAnimations) {
              	//需要执行预测动画，设置 InPreLayout 为 true
                mState.mInPreLayout = true;                                                     
            } else {                                                                            
                // consume remaining updates to provide a consistent state with the layout pass.
                mAdapterHelper.consumeUpdatesInOnePass();                                       
                mState.mInPreLayout = false;                                                    
            }                                                                                   
            mAdapterUpdateDuringMeasure = false;                                                
            resumeRequestLayout(false);                                                         
        } else if (mState.mRunPredictiveAnimations) {                                           
            //这意味着已经有一个 onMeasuer 被调用，来处理等待的 adapter change，如果 RecyclerView 是作为 LinearLayout 的 child view，并且 layout_width = MATCH_PARENT，可能会有两个 onMeasure 被调用。RecyclerView 不能第二次调用 LayoutManager.onMeasure()，因为当 LayoutManager 使用 child 去测量时，getViewForPosition() 将奔溃。
            setMeasuredDimension(getMeasuredWidth(), getMeasuredHeight());                      
            return;                                                                             
        }                                                                                       
                                                                                                
        if (mAdapter != null) {                                                                 
            mState.mItemCount = mAdapter.getItemCount();                                        
        } else {                                                                                
            mState.mItemCount = 0;                                                              
        }                                                                                       
        eatRequestLayout();                                                                     
      	//LayoutManager.onMeasure
        mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);                            
        resumeRequestLayout(false);                                                             
        mState.mInPreLayout = false; // clear                                                   
    }                                                                                           
}                                                                                               

/**                                                                                            
 * Used when onMeasure is called before layout manager is set                                  
 */                                                                                            
void defaultOnMeasure(int widthSpec, int heightSpec) {                                         
    // calling LayoutManager here is not pretty but that API is already public and it is better
    // than creating another method since this is internal.                                    
    final int width = LayoutManager.chooseSize(widthSpec,                                      
            getPaddingLeft() + getPaddingRight(),                                              
            ViewCompat.getMinimumWidth(this));                                                 
    final int height = LayoutManager.chooseSize(heightSpec,                                    
            getPaddingTop() + getPaddingBottom(),                                              
            ViewCompat.getMinimumHeight(this));                                                
                                                                                               
    setMeasuredDimension(width, height);                                                       
}                                                                                              
```

``` java
/**                                                                                             
 * The first step of a layout where we;                                                         
 * - process adapter updates                                                                    
 * - decide which animation should run                                                          
 * - save information about current views                                                       
 * - If necessary, run predictive layout and save its information                               
 */                                                                                             
private void dispatchLayoutStep1() {                                                            
  	//校验状态值
    mState.assertLayoutStep(State.STEP_START);                                                  
	//记录剩余的滚动值
    fillRemainingScrollValues(mState);                                                          
    mState.mIsMeasuring = false;                                                                
  	//进入 layout 步骤 flags
    eatRequestLayout();
  	//清除 view 动画缓存
    mViewInfoStore.clear();                                                                     
  	//设置进入 layout 步骤标记，mLayoutOrScrollCounter++;
    onEnterLayoutOrScroll();                                                                    
  	//根据 Adapter 的更新来计算我们想要运行的动画类型
    processAdapterUpdatesAndSetAnimationFlags();
  	//保存当前获取焦点的 ViewHolder 信息
    saveFocusInfo();                                                                            
  	//更新 State
    mState.mTrackOldChangeHolders = mState.mRunSimpleAnimations && mItemsChanged;               
    mItemsAddedOrRemoved = mItemsChanged = false;                                               
    mState.mInPreLayout = mState.mRunPredictiveAnimations;                                      
    mState.mItemCount = mAdapter.getItemCount();                                                
  	//获取已添加到 RecyclerView 的 view 的最大和最小 LayoutPosition
    findMinMaxChildLayoutPositions(mMinMaxLayoutPositions);                                     
                                                                                                
    if (mState.mRunSimpleAnimations) {                                                          
      	//需要执行动画
        // Step 0: Find out where all non-removed items are, pre-layout                         
        int count = mChildHelper.getChildCount();                                               
        for (int i = 0; i < count; ++i) {                                                       
            final ViewHolder holder = getChildViewHolderInt(mChildHelper.getChildAt(i));        
            if (holder.shouldIgnore() || (holder.isInvalid() && !mAdapter.hasStableIds())) {    
                continue;                                                                       
            }                                                                                   
          	//调用 ItemAnimator.recordPreLayoutInfomation，记录运行动画所需的信息
            final ItemHolderInfo animationInfo = mItemAnimator                                  
                    .recordPreLayoutInformation(mState, holder,                                 
                            ItemAnimator.buildAdapterChangeFlagsForAnimations(holder),          
                            holder.getUnmodifiedPayloads());                                    
          	//保存 view 动画缓存
            mViewInfoStore.addToPreLayout(holder, animationInfo);                               
            if (mState.mTrackOldChangeHolders && holder.isUpdated() && !holder.isRemoved()      
                    && !holder.shouldIgnore() && !holder.isInvalid()) {                         
                long key = getChangedHolderKey(holder);                                         
                // 这不是唯一将 ViewHolder 添加到 oldChangeHolders 的地方。
              	// * ViewHoldr 当前隐藏或者没被删除。
              	// * 在 Adapter 中的 隐藏的 item 被改变。
              	// * LayoutManager 决定在 pre-layout 步骤中 layout item。
              	//将给定的 ViewHolder 添加到 oldChangeHolders
                mViewInfoStore.addToOldChangeHolders(key, holder);                              
            }                                                                                   
        }                                                                                       
    }                                                                                           
    if (mState.mRunPredictiveAnimations) {                                                      
                                                                                         
        //第一步：运行 pre-layout：这将使用 old positions of items。LayoutMananger 将布置一切，甚至包括删除 item（尽管不会将删除的 item 添加回容器）。这给出了作为 real layout 的一部分 APPEARING views 的 pre-layout position。
      	// 保存 old positions，所以 LayoutManager 可以执行它的映射逻辑。
        saveOldPositions();                                                                     
        final boolean didStructureChange = mState.mStructureChanged;                            
        mState.mStructureChanged = false;                                                       
      	//暂时禁用 mStructureChanged flag，因为我们要求之前的布局
        mLayout.onLayoutChildren(mRecycler, mState);                                            
        mState.mStructureChanged = didStructureChange;                                          
                                                                                                
        for (int i = 0; i < mChildHelper.getChildCount(); ++i) {                                
            final View child = mChildHelper.getChildAt(i);                                      
            final ViewHolder viewHolder = getChildViewHolderInt(child);                         
            if (viewHolder.shouldIgnore()) {                                                    
                continue;                                                                       
            }                                                                                   
            if (!mViewInfoStore.isInPreLayout(viewHolder)) {                                    
              	//ViewHolder 不在 pre-layout 中
              	//构建 change flags
                int flags = ItemAnimator.buildAdapterChangeFlagsForAnimations(viewHolder);      
                boolean wasHidden = viewHolder                                                  
                        .hasAnyOfTheFlags(ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST);            
                if (!wasHidden) {
                  	//当前 ViewHolder 为隐藏状态
                    flags |= ItemAnimator.FLAG_APPEARED_IN_PRE_LAYOUT;                          
                }                                                                               
              	//调用 ItemAnimator.recordPreLayoutInfomation，记录运行动画所需的信息
                final ItemHolderInfo animationInfo = mItemAnimator.recordPreLayoutInformation(  
                        mState, viewHolder, flags, viewHolder.getUnmodifiedPayloads());         
                if (wasHidden) {                                                                
                  	//当前为隐藏状态，记录从隐藏列表中出现的动画信息，并且清除隐藏标记。
                    recordAnimationInfoIfBouncedHiddenView(viewHolder, animationInfo);          
                } else {
                  	//将 ViewHolder 添加到已出现的 pre-layout 列表中。
                    mViewInfoStore.addToAppearedInPreLayoutHolders(viewHolder, animationInfo);  
                }                                                                               
            }                                                                                   
        }                                                                                       
        //不处理正在消失的 ViewHolder，因为它们可能通过 post layout 重新出现。
      	//清除 old positions
        clearOldPositions();                                                                    
    } else {                                                                                    
        clearOldPositions();                                                                    
    }
  	//退出 layout：mLayoutOrScrollCounter--
    onExitLayoutOrScroll();                                                                     
  	//重新开始 request layout
    resumeRequestLayout(false);                                                                 
    mState.mLayoutStep = State.STEP_LAYOUT;                                                     
}                                                                                               
```

``` java
/**                                                                                      
 * The second layout step where we do the actual layout of the views for the final state.
 * This step might be run multiple times if necessary (e.g. measure).                    
 */                                                                                      
private void dispatchLayoutStep2() {
  	//和步骤 1 一样，校验，设置 layout flags
    eatRequestLayout();                                                                  
    onEnterLayoutOrScroll();                                                             
    mState.assertLayoutStep(State.STEP_LAYOUT | State.STEP_ANIMATIONS); 
  	//一次性消费所有更新操作。
    mAdapterHelper.consumeUpdatesInOnePass();
  	//更新 state
    mState.mItemCount = mAdapter.getItemCount();                                         
    mState.mDeletedInvisibleItemCountSincePreviousLayout = 0;                            
                                                                                         
    // 执行 layout                                                                
    mState.mInPreLayout = false;                                                         
    mLayout.onLayoutChildren(mRecycler, mState);                                         
                                                                                         
    mState.mStructureChanged = false;                                                    
    mPendingSavedState = null;                                                           
                                                                                         
    // onLayoutChildren 可能有禁用 item animations；重新检查
    mState.mRunSimpleAnimations = mState.mRunSimpleAnimations && mItemAnimator != null;  
    mState.mLayoutStep = State.STEP_ANIMATIONS;
  	//退出 layout 处理
    onExitLayoutOrScroll();                                                              
    resumeRequestLayout(false);                                                          
}                                                                                        
```

### Layout

``` java
@Override                                                             
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    TraceCompat.beginSection(TRACE_ON_LAYOUT_TAG);                    
    dispatchLayout();                                                 
    TraceCompat.endSection();                                         
    mFirstLayoutComplete = true;                                      
}

/**
* 包装 layoutChildren() 去处理 layout 导致的动画变化。
* 动画假定工作在以下五种不同的情况：
* PERSISTENT: item 在 layout 之前和之后都可见。
* REMOVED: item 在 layout 之前是可见，之后被应用删除。
* ADDED: item 在 layout 之前是不存在的，之后被应用添加。
* DISAPPEARING: item 在 layout 前后都在数据集中存在，但在 layout 处理中，从可见到不可见的变化（从屏幕中移开，作为其他改变的附带效果）。
* APPEARING: item 在 layout 前后都在数据集中存在，但在 layout 处理中，从不可见到可见的变化（移动到屏幕中，作为其他改变的附带效果）。
* 通过计算 layout 前后存在哪些 items，并推断每个 item 是五种状态中的哪一种，然后设置相应的动画：
* PERSISTENT views 调用 animatePersistence
* DISAPPEARING views 调用 animateDisappearance
* APPEARING views 调用 animateAppearance
* 改变的 views 调用 animateChange
*/
void dispatchLayout() {                                                                       
    if (mAdapter == null) {                                                                   
        Log.e(TAG, "No adapter attached; skipping layout");                                   
        // leave the state in START                                                           
        return;                                                                               
    }                                                                                         
    if (mLayout == null) {                                                                    
        Log.e(TAG, "No layout manager attached; skipping layout");                            
        // leave the state in START                                                           
        return;                                                                               
    }                                                                                         
    mState.mIsMeasuring = false;                                                              
    if (mState.mLayoutStep == State.STEP_START) {
      	//当前为 STEP_START
      	//依次调用 dispatchLayoutStep1 和 dispatchLayoutStep2
        dispatchLayoutStep1();                                                                
        mLayout.setExactMeasureSpecsFrom(this);                                               
        dispatchLayoutStep2();                                                                
    } else if (mAdapterHelper.hasUpdates() || mLayout.getWidth() != getWidth()                
            || mLayout.getHeight() != getHeight()) {                                          
        //前两个步骤都在 onMeasure 中完成了，但我们应该重新运行来处理尺寸的改变。                                                                   
        mLayout.setExactMeasureSpecsFrom(this);                                               
        dispatchLayoutStep2();                                                                
    } else {                                                                                  
        // always make sure we sync them (to ensure mode is exact)                            
        mLayout.setExactMeasureSpecsFrom(this);                                               
    }                                                                                         
    dispatchLayoutStep3();                                                                    
}                                                                                             
```

``` java
/**                                                                                              
* layout 的最后一步，我们保存关于 view 动画的信息，触发动画，并且执行并要的清理。
*/
private void dispatchLayoutStep3() {                                                             
  	//必要校验和 flags
    mState.assertLayoutStep(State.STEP_ANIMATIONS);                                              
    eatRequestLayout();                                                                          
    onEnterLayoutOrScroll();                                                                     
    mState.mLayoutStep = State.STEP_START;                                                       
    if (mState.mRunSimpleAnimations) {                                                           
      	//步骤 3:找到现在的视图情况，并且处理改变动画。
      	//反向遍历列表，因为我们在循环中可能调用 animateChange，这可能会删除目标 view holder。
        for (int i = mChildHelper.getChildCount() - 1; i >= 0; i--) {                            
            ViewHolder holder = getChildViewHolderInt(mChildHelper.getChildAt(i));               
            if (holder.shouldIgnore()) {                                                         
                continue;                                                                        
            }                                                                                    
            long key = getChangedHolderKey(holder);                                              
          	//在 layout 完成后调用，让 ItemAnimator 保存 view 最终状态的必要信息。
            final ItemHolderInfo animationInfo = mItemAnimator                                   
                    .recordPostLayoutInformation(mState, holder);                                
            ViewHolder oldChangeViewHolder = mViewInfoStore.getFromOldChangeHolders(key);        
            if (oldChangeViewHolder != null && !oldChangeViewHolder.shouldIgnore()) {            
				// 执行一个改变动画。
               	// 如果一个 Item 是 CHANGED，但是在更新后的版本是不出现，那这是矛盾的。
              	// 由于被标记为 DISAPPEARING 的 view 很有可能会超出其范围，我们执行改变动画。一旦它们的动画结束时，两个 views 将被自动清理。
              	// 在其他的处理中，如果同一个 view holder 实例，我们执行一个消失的动画去代替，因为我们将不会去 rebind 更新 ViewHolder，直到通过 LayoutMananger 强制执行。
                final boolean oldDisappearing = mViewInfoStore.isDisappearing(                   
                        oldChangeViewHolder);                                                    
                final boolean newDisappearing = mViewInfoStore.isDisappearing(holder);           
                if (oldDisappearing && oldChangeViewHolder == holder) {                          
                  	//执行消失动画，而不是改变动画。
                    mViewInfoStore.addToPostLayout(holder, animationInfo);                       
                } else {                                                                         
                  	// oldDisappearing = false 或 oldChangeViewHolder 不等于 holder。
                    final ItemHolderInfo preInfo = mViewInfoStore.popFromPreLayout(              
                            oldChangeViewHolder);                                                
                    // we add and remove so that any post info is merged.                        
                    mViewInfoStore.addToPostLayout(holder, animationInfo);                       
                    ItemHolderInfo postInfo = mViewInfoStore.popFromPostLayout(holder);          
                    if (preInfo == null) {                                                       
                      	//不能找到之前的 view 信息，当作错误处理。
                        handleMissingPreInfoForChangeError(key, holder, oldChangeViewHolder);    
                    } else {                                                                     
                      	//正常的改变情况，可见 -> 不可见，不可见 -> 可见
                        animateChange(oldChangeViewHolder, holder, preInfo, postInfo,            
                                oldDisappearing, newDisappearing);                               
                    }                                                                            
                }                                                                                
            } else {                                                                             
              	//layout 之前不存在。
                mViewInfoStore.addToPostLayout(holder, animationInfo);                           
            }                                                                                    
        }                                                                                        
                                                                                                 		// 步骤 4：处理 view info 列表，并且触发动画。
        mViewInfoStore.process(mViewInfoProcessCallback);                                        
    }                                                                                            
  	//删除并且回收废弃的 ViewHolder。
    mLayout.removeAndRecycleScrapInt(mRecycler);
  	//保存，并且清理
    mState.mPreviousLayoutItemCount = mState.mItemCount;                                       
    mDataSetHasChangedAfterLayout = false;                                                     
    mState.mRunSimpleAnimations = false;                                                       
                                                                                               
    mState.mRunPredictiveAnimations = false;                                                   
    mLayout.mRequestedSimpleAnimations = false;                                                
    if (mRecycler.mChangedScrap != null) {                                                     
        mRecycler.mChangedScrap.clear();                                                       
    }                                                                                          
    if (mLayout.mPrefetchMaxObservedInInitialPrefetch) {                                       
        // Initial prefetch has expanded cache, so reset until next prefetch.                  
        // This prevents initial prefetches from expanding the cache permanently.              
        mLayout.mPrefetchMaxCountObserved = 0;                                                 
        mLayout.mPrefetchMaxObservedInInitialPrefetch = false;                                 
        mRecycler.updateViewCacheSize();                                                       
    }                                                                                          
    //调用 LayoutManager.onLayoutCompleted()                                                                                           
    mLayout.onLayoutCompleted(mState);                                                         
    onExitLayoutOrScroll();                                                                    
    resumeRequestLayout(false);                                                                
    mViewInfoStore.clear();                                                                    
    if (didChildRangeChange(mMinMaxLayoutPositions[0], mMinMaxLayoutPositions[1])) {
      	// layout 过程进行了滚动操作，分发滚动改变事件。
        dispatchOnScrolled(0, 0);                                                              
    }
  	//恢复 dispatchLayoutStep1() 中设置的焦点视图。
    recoverFocusFromState();                                                                   
    resetFocusInfo();                                                                          
}                                                                                              
```

### Draw

``` java
@Override                                                      
public void onDraw(Canvas c) {                                 
    super.onDraw(c);                                           
                                                               
    final int count = mItemDecorations.size();                 
    for (int i = 0; i < count; i++) {
    	//绘制 ItemDecorations
        mItemDecorations.get(i).onDraw(c, this, mState);       
    }                                                          
}                                                              
```

