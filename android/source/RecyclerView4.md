## Recycler

> 基于 LinearLayoutManager 分析。

### Get

调用链：

LinearLayoutManager.layoutChunk => LayoutState.next => Recycler.getViewForPosition => 		  Recycler.tryGetViewHolderForPositionByDeadline

``` java
/**                                                                                        
 * 尝试根据 position 获取 ViewHoler，无论是从 Recycler scrap，cache，RecyclerdViewPool，或者 直接创建。                                  
 * <p>                                                                                     
 * 如果传递 FOREVER_NS 以外的 deadlineNs，如果它认为时间不够，则这个方法会尽早返回，而不是创建或绑定 ViewHolder。如果 ViewHolder 必须创建，但没有足够的时间，则它将返回 null。如果获取了 ViewHolder，并且它必须被绑定，但没有足够时间，那么将返回一个没有绑定的 ViewHolder，在返回的 ViewHolder 上使用 isBound 方法可以检查。
 *                                                                                         
 * @param position   需要返回的 ViewHolder 的 position                                
 * @param dryRun     如果为 true,则 ViewHoler 不应该从 scrap/cache 中删除。        
 * @param deadlineNs 时间，相对于 getNanoTime()，在这个时间内，bind/create 工作应该完成。如果传递了 FOREVER_NS，那么如果需要，这个方法 create/bind holder 将不会出错。                                    
 *                                                                                                                                     
 */                                                                                        
@Nullable                                                                                  
ViewHolder tryGetViewHolderForPositionByDeadline(int position,                             
        boolean dryRun, long deadlineNs) {                                                 
    if (position < 0 || position >= mState.getItemCount()) {                               
        throw new IndexOutOfBoundsException("Invalid item position " + position            
                + "(" + position + "). Item count:" + mState.getItemCount()                
                + exceptionLabel());                                                       
    }                                                                                      
    boolean fromScrapOrHiddenOrCache = false;                                              
    ViewHolder holder = null;                                                              
    // 0) 如果存在 changed scrap，从中寻找。                             
    if (mState.isPreLayout()) {                                                            
        holder = getChangedScrapViewForPosition(position);                                 
        fromScrapOrHiddenOrCache = holder != null;                                         
    }                                                                                      
    // 1) 通过 position，从 attach scrap，hidden child 或者 cache 中寻找。                              
    if (holder == null) {                                                                  
        holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun);              
        if (holder != null) {
          	// 检查 viewholder 是否可用于指定 position。
            if (!validateViewHolderForOffsetPosition(holder)) {                            
                // 如果当前 viewholer 还不能用，recycle holder （相应的 unscrap）。         
                if (!dryRun) {                                                             
                    // 我们想要去 recycle 它，但是需要确保它没有被用于动画逻辑等等。                                              
                    holder.addFlags(ViewHolder.FLAG_INVALID);                              
                  	//如果需要 unscrap
                    if (holder.isScrap()) {                                                
                        removeDetachedView(holder.itemView, false);                        
                        holder.unScrap();                                                  
                    } else if (holder.wasReturnedFromScrap()) {                            
                        holder.clearReturnedFromScrapFlag();                               
                    }                 
                  	// recycle view holder
                    recycleViewHolderInternal(holder);                                     
                }                                                                          
                holder = null;                                                             
            } else {
              	// 标记来源
                fromScrapOrHiddenOrCache = true;                                           
            }                                                                              
        }                                                                                  
    }                                                                                      
    if (holder == null) {                                                                  
        final int offsetPosition = mAdapterHelper.findPositionOffset(position);            
        if (offsetPosition < 0 || offsetPosition >= mAdapter.getItemCount()) {             
            throw new IndexOutOfBoundsException("Inconsistency detected. Invalid item "    
                    + "position " + position + "(offset:" + offsetPosition + ")."          
                    + "state:" + mState.getItemCount() + exceptionLabel());                
        }                                                                                  
                                                                                           
        final int type = mAdapter.getItemViewType(offsetPosition);                         
        // 2) 如果它存在 stable ids，通过它从 scrap/cache 中寻找。                            
        if (mAdapter.hasStableIds()) {                                                     
            holder = getScrapOrCachedViewForId(mAdapter.getItemId(offsetPosition),         
                    type, dryRun);                                                         
            if (holder != null) {                                                          
                // 更新 position                                                  
                holder.mPosition = offsetPosition;                                         
                fromScrapOrHiddenOrCache = true;                                           
            }                                                                              
        }                                                                                  
        if (holder == null && mViewCacheExtension != null) {
          	// 从扩展缓存中获取。
            // We are NOT sending the offsetPosition because LayoutManager does not        
            // know it.                                                                    
            final View view = mViewCacheExtension                                          
                    .getViewForPositionAndType(this, position, type);                      
            if (view != null) {                                                            
                holder = getChildViewHolder(view);                                         
                if (holder == null) {                                                      
                    throw new IllegalArgumentException("getViewForPositionAndType returned"
                            + " a view which does not have a ViewHolder"                   
                            + exceptionLabel());                                           
                } else if (holder.shouldIgnore()) {                                        
                    throw new IllegalArgumentException("getViewForPositionAndType returned"
                            + " a view that is ignored. You must call stopIgnoring before" 
                            + " returning this view." + exceptionLabel());                 
                }                                                                          
            }                                                                              
        }                                                                                  
        if (holder == null) { // fallback to pool                                          
            if (DEBUG) {                                                                   
                Log.d(TAG, "tryGetViewHolderForPositionByDeadline("                        
                        + position + ") fetching from shared pool");                       
            }                                                                              
            holder = getRecycledViewPool().getRecycledView(type);                          
            if (holder != null) {                                                          
                holder.resetInternal();                                                    
                if (FORCE_INVALIDATE_DISPLAY_LIST) {
                  	// SDK_INT = 18，19，20 需要通过设置 view 为 INVISIBLE，来使得 DisplayList 失效。
                   //在 18，19，20 上如果一个 view 的层级有两层的，就会阻止 DisplayList 失效。
                    invalidateDisplayListInt(holder);                                      
                }                                                                          
            }                                                                              
        }                                                                                  
        if (holder == null) {                                                              
            long start = getNanoTime();                                                    
            if (deadlineNs != FOREVER_NS                                                   
                    && !mRecyclerPool.willCreateInTime(type, start, deadlineNs)) {         
                // abort - we have a deadline we can't meet
              	// 如果不能在截至时间之前完成，返回 null。
                return null;                                                               
            }
          	//调用 Adapter.createViewHolder
            holder = mAdapter.createViewHolder(RecyclerView.this, type);                   
            if (ALLOW_THREAD_GAP_WORK) {
              	// 嵌套的 RecyclerView
                // only bother finding nested RV if prefetching                            
                RecyclerView innerView = findNestedRecyclerView(holder.itemView);          
                if (innerView != null) {                                                   
                    holder.mNestedRecyclerView = new WeakReference<>(innerView);           
                }                                                                          
            }                                                                              
                                                                                           
            long end = getNanoTime();
          	// 计算指定 type ViewHolder 创建平均时间。
            mRecyclerPool.factorInCreateTime(type, end - start);                           
            if (DEBUG) {                                                                   
                Log.d(TAG, "tryGetViewHolderForPositionByDeadline created new ViewHolder");
            }                                                                              
        }                                                                                  
    }                                                                                      
                                                                                           
    // This is very ugly but the only place we can grab this information                   
    // before the View is rebound and returned to the LayoutManager for post layout ops.   
    // We don't need this in pre-layout since the VH is not updated by the LM.             
    if (fromScrapOrHiddenOrCache && !mState.isPreLayout() && holder                        
            .hasAnyOfTheFlags(ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST)) {                 
        holder.setFlags(0, ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST);                      
        if (mState.mRunSimpleAnimations) {                                                 
            int changeFlags = ItemAnimator                                                 
                    .buildAdapterChangeFlagsForAnimations(holder);                         
            changeFlags |= ItemAnimator.FLAG_APPEARED_IN_PRE_LAYOUT;                       
            final ItemHolderInfo info = mItemAnimator.recordPreLayoutInformation(mState,   
                    holder, changeFlags, holder.getUnmodifiedPayloads());                  
            recordAnimationInfoIfBouncedHiddenView(holder, info);                          
        }                                                                                  
    }                                                                                      
                                                                                           
    boolean bound = false;                                                                 
    if (mState.isPreLayout() && holder.isBound()) {                                        
        // do not update unless we absolutely have to.                                     
        holder.mPreLayoutPosition = position;                                              
    } else if (!holder.isBound() || holder.needsUpdate() || holder.isInvalid()) {          
        if (DEBUG && holder.isRemoved()) {                                                 
            throw new IllegalStateException("Removed holder should be bound and it should" 
                    + " come here only in pre-layout. Holder: " + holder                   
                    + exceptionLabel());                                                   
        }                                                                                  
        final int offsetPosition = mAdapterHelper.findPositionOffset(position);
      	//尝试在截至时间之前完成 bind view holder 操作。
        bound = tryBindViewHolderByDeadline(holder, offsetPosition, position, deadlineNs); 
    }                                                                                      
     
  	// layoutParams
    final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();                   
    final LayoutParams rvLayoutParams;                                                     
    if (lp == null) {                                                                      
        rvLayoutParams = (LayoutParams) generateDefaultLayoutParams();                     
        holder.itemView.setLayoutParams(rvLayoutParams);                                   
    } else if (!checkLayoutParams(lp)) {                                                   
        rvLayoutParams = (LayoutParams) generateLayoutParams(lp);                          
        holder.itemView.setLayoutParams(rvLayoutParams);                                   
    } else {                                                                               
        rvLayoutParams = (LayoutParams) lp;                                                
    }                                                                                      
    rvLayoutParams.mViewHolder = holder;
  	// 等待调用 invalidate，如果 bound，并且 viewholer 是从 scrap/hidden/caceh 中获取的。
    rvLayoutParams.mPendingInvalidate = fromScrapOrHiddenOrCache && bound;                 
    return holder;                                                                         
}                                                                                          
```

``` java
/**                                                                                              
 * Attempts to bind view, and account for relevant timing information. If                        
 * deadlineNs != FOREVER_NS, this method may fail to bind, and return false.                     
 *                                                                                               
 * @param holder Holder to be bound.                                                             
 * @param offsetPosition Position of item to be bound.                                           
 * @param position Pre-layout position of item to be bound.                                      
 * @param deadlineNs Time, relative to getNanoTime(), by which bind/create work should           
 *                   complete. If FOREVER_NS is passed, this method will not fail to             
 *                   bind the holder.                                                            
 * @return                                                                                       
 */                                                                                              
private boolean tryBindViewHolderByDeadline(ViewHolder holder, int offsetPosition,               
        int position, long deadlineNs) {                                                         
    holder.mOwnerRecyclerView = RecyclerView.this;                                               
    final int viewType = holder.getItemViewType();                                               
    long startBindNs = getNanoTime();                                                            
    if (deadlineNs != FOREVER_NS                                                                 
            && !mRecyclerPool.willBindInTime(viewType, startBindNs, deadlineNs)) {               
        // 如果不能在截至时间之前完成 bind，返回 null。                                              
        return false;                                                                            
    }
  	// 调用 adapter.bindViewHoler
    mAdapter.bindViewHolder(holder, offsetPosition);                                             
    long endBindNs = getNanoTime();
  	// 计算 bind view holer 平均时间
    mRecyclerPool.factorInBindTime(holder.getItemViewType(), endBindNs - startBindNs);           
    attachAccessibilityDelegateOnBind(holder);                                                   
    if (mState.isPreLayout()) {                                                                  
        holder.mPreLayoutPosition = position;                                                    
    }                                                                                            
    return true;                                                                                 
}                                                                                                
```

获取 recycle view 的优先级如下所示：

0. 如果 `pre-layout = true`，则从 `Recycler.mChangedScrap` 中获取。
1. 先从 `Recycler.mAttachedScrap` 中根据 position 获取，如果不存在，当 `dryRun = false` 则从 `ChildHelper.mHiddenViews` 中获取。如果不存在，则从 `Recycler.mCachedViews` 中获取。
2. 如果 `Adapter.hasStableIds() = true`，则通过 id 从 `Recycler.mAttachedScrap` 中获取。如果不存在，当 `ViewCacheExtension != null`，则从 ViewCacheExtension 中获取。如果不存在，则从 `RecyclerPool` 中获取。

以上步骤后，还没能获取 viewholder，则调用 `Adapter.onCreateViewHoler`。

### Recycle

``` java
// LayoutManager
private void scrapOrRecycleView(Recycler recycler, int index, View view) { 
    final ViewHolder viewHolder = getChildViewHolderInt(view);             
    if (viewHolder.shouldIgnore()) {                                       
        if (DEBUG) {                                                       
            Log.d(TAG, "ignoring view " + viewHolder);                     
        }                                                                  
        return;                                                            
    }                                                                      
    if (viewHolder.isInvalid() && !viewHolder.isRemoved()                  
            && !mRecyclerView.mAdapter.hasStableIds()) {
      	// viewholer 有效，viewholer 没有被删除，并且 adapter 拥有稳定的 ids。
      	// remove	
        removeViewAt(index);                                               
      	// recycle
        recycler.recycleViewHolderInternal(viewHolder);                    
    } else {
      	// detach
        detachViewAt(index);
      	// scrap
        recycler.scrapView(view);
      	// 从 disappearing list 中删除这个 viewholer
        mRecyclerView.mViewInfoStore.onViewDetached(viewHolder);           
    }                                                                      
}                                                                          
```

#### recycle

``` java
/**
 * Recycler
 * 优先缓存到 cached，否则放到 RecyclerViewPool 
**/
void recycleViewHolderInternal(ViewHolder holder) {                                           
    if (holder.isScrap() || holder.itemView.getParent() != null) {
      	// 判断是否已经 scrap
        throw new IllegalArgumentException(                                                   
                "Scrapped or attached views may not be recycled. isScrap:"                    
                        + holder.isScrap() + " isAttached:"                                   
                        + (holder.itemView.getParent() != null) + exceptionLabel());          
    }                                                                                         
                                                                                              
    if (holder.isTmpDetached()) {
      	// 判断是否已经 detached
        throw new IllegalArgumentException("Tmp detached view should be removed "             
                + "from RecyclerView before it can be recycled: " + holder                    
                + exceptionLabel());                                                          
    }                                                                                         
                                                                                              
    if (holder.shouldIgnore()) {
      	// 判断是否应该 ignore
        throw new IllegalArgumentException("Trying to recycle an ignored view holder. You"    
                + " should first call stopIgnoringView(view) before calling recycle."         
                + exceptionLabel());                                                          
    }                                                                                         
    // 判断是否有 transient state，可能处于动画期间                                                                 
    final boolean transientStatePreventsRecycling = holder                                    
            .doesTransientStatePreventRecycling();
  	// 回调 Adapter.onFailedToRecycleView 是否能够强制 recycle
    final boolean forceRecycle = mAdapter != null                                             
            && transientStatePreventsRecycling                                                
            && mAdapter.onFailedToRecycleView(holder);                                        
    boolean cached = false;                                                                   
    boolean recycled = false;                                                                 
    if (DEBUG && mCachedViews.contains(holder)) {                                             
        throw new IllegalArgumentException("cached view received recycle internal? "          
                + holder + exceptionLabel());                                                 
    }                                                                                         
    if (forceRecycle || holder.isRecyclable()) {
      	// 可以进行 recycle
        if (mViewCacheMax > 0                                                                 
                && !holder.hasAnyOfTheFlags(ViewHolder.FLAG_INVALID                           
                | ViewHolder.FLAG_REMOVED                                                     
                | ViewHolder.FLAG_UPDATE                                                      
                | ViewHolder.FLAG_ADAPTER_POSITION_UNKNOWN)) {
          	// 优先淘汰最早的 cached view
            int cachedViewSize = mCachedViews.size();                                         
            if (cachedViewSize >= mViewCacheMax && cachedViewSize > 0) {                      
                recycleCachedViewAt(0);                                                       
                cachedViewSize--;                                                             
            }                                                                                 
                                                                                              
            int targetCacheIndex = cachedViewSize;                                            
            if (ALLOW_THREAD_GAP_WORK                                                         
                    && cachedViewSize > 0                                                     
                    && !mPrefetchRegistry.lastPrefetchIncludedPosition(holder.mPosition)) {   
                // when adding the view, skip past most recently prefetched views             
                int cacheIndex = cachedViewSize - 1;                                          
                while (cacheIndex >= 0) {                                                     
                    int cachedPos = mCachedViews.get(cacheIndex).mPosition;                   
                    if (!mPrefetchRegistry.lastPrefetchIncludedPosition(cachedPos)) {         
                        break;                                                                
                    }                                                                         
                    cacheIndex--;                                                             
                }                                                                             
                targetCacheIndex = cacheIndex + 1;                                            
            }
          	// 将 viewholer 添加得到 cached 中
            mCachedViews.add(targetCacheIndex, holder);                                       
            cached = true;                                                                    
        }                                                                                     
        if (!cached) {
          	// 如果没有添加到 cached 中，则添加到 RecyclerViewPool
            addViewHolderToRecycledViewPool(holder, true);                                    
            recycled = true;                                                                  
        }                                                                                     
    } else {
      	// 记住：当一个 view 在动画运行中滚动，那么它可能会 recycled 出错。在这种情况下，item 最终在 ItemAnimatorRestoreListener.onAnimationFinished 中 recycled                                                                          
        // TODO: consider cancelling an animation when an item is removed scrollBy,           
        // to return it to the pool faster                                                    
        if (DEBUG) {                                                                          
            Log.d(TAG, "trying to recycle a non-recycleable holder. Hopefully, it will "      
                    + "re-visit here. We are still removing it from animation lists"          
                    + exceptionLabel());                                                      
        }                                                                                     
    }                                                                                         
    // 即使 holder 没有被删除，我们让调用这个方法让它从 viewHoler list 中删除。                                                               
    mViewInfoStore.removeViewHolder(holder);                                                  
    if (!cached && !recycled && transientStatePreventsRecycling) {
      	// 如果没有 cached，recycled，并且拥有 transient state
        holder.mOwnerRecyclerView = null;                                                     
    }                                                                                         
}                                                                                             
```

``` java
/**
 * Recycler	
 * Prepares the ViewHolder to be removed/recycled, and inserts it into the RecycledViewPool.
 *                                                                                          
 * Pass false to dispatchRecycled for views that have not been bound.                       
 *                                                                                          
 * @param holder Holder to be added to the pool.                                            
 * @param dispatchRecycled True to dispatch View recycled callbacks.                        
 */                                                                                         
void addViewHolderToRecycledViewPool(ViewHolder holder, boolean dispatchRecycled) {
  	// 清除嵌套 RecyclerView
    clearNestedRecyclerViewIfNotNested(holder);                                             
    if (holder.hasAnyOfTheFlags(ViewHolder.FLAG_SET_A11Y_ITEM_DELEGATE)) {                  
        holder.setFlags(0, ViewHolder.FLAG_SET_A11Y_ITEM_DELEGATE);                         
        ViewCompat.setAccessibilityDelegate(holder.itemView, null);                         
    }
  	// 分发回调
    if (dispatchRecycled) {                                                                 
        dispatchViewRecycled(holder);                                                       
    }                                                                                       
    holder.mOwnerRecyclerView = null;
  	// 放入 RecyclerViewPool
    getRecycledViewPool().putRecycledView(holder);                                          
}                                                                                           
```

#### scrap

``` java
/* 
 * LayoutManager
 * 暂时 detach child view                                                        
 *                                                                                         
 * <p> LayoutManager 可能希望执行轻量级的 detach 操作来重新排列当前 attached 到 	  RecyclerView 的 view。通常 LayoutManager 实现会使用 detachAndScrapView 方法，所以  detached view 可能需要 rebound 和 resused。</p>                                 
 *                                                                                         
 * <p>如果 LayoutManager 使用这个方法去 detach child,那在由 RecyclerView 调用 LayoutManager 的入口点方法返回前，LayoutManager 必须调用 attachView 方法来 reattcah 或者 调用 removeDetachedView 去完全删除，这个 detached view。
 */
public void detachViewAt(int index) {                                                      
    detachViewInternal(index, getChildAt(index));                                          
}                                                                                          
                                                                                           
private void detachViewInternal(int index, View view) {                                    
    if (DISPATCH_TEMP_DETACH) {                                                            
        ViewCompat.dispatchStartTemporaryDetach(view);                                     
    }                                                                                      
    mChildHelper.detachViewFromParent(index);                                              
}                                                                                          
```

``` java
 /**                                                                                         
  * 标记一个 attached view 为 scrap。
  * 如果 viewholer 不需要更新，则添加到 attach scrap，否则添加到 changed scrap
  *                                                                                          
  * <p>"Scrap" views 仍然 attached 在 RecyclerView，但依然可以 rebinding 和 reuse。
  * 通过给定的 position 去请求一个 view，可能会返回一个 reused 或者 rebound 的 scrap view 
  * </p>                                               
  *                                                                                                                                                         
  */                                                                                         
 void scrapView(View view) {                                                                 
     final ViewHolder holder = getChildViewHolderInt(view);                                  
     if (holder.hasAnyOfTheFlags(ViewHolder.FLAG_REMOVED | ViewHolder.FLAG_INVALID)          
             || !holder.isUpdated() || canReuseUpdatedViewHolder(holder)) {                  
         if (holder.isInvalid() && !holder.isRemoved() && !mAdapter.hasStableIds()) {        
             throw new IllegalArgumentException("Called scrap view with an invalid view."    
                     + " Invalid views cannot be reused from scrap, they should rebound from"
                     + " recycler pool." + exceptionLabel());                                
         }
       	 // 如果 holder 标记为 删除并且无效，或者没有更新，或者可以 reused
         // 设置为 scrap
         holder.setScrapContainer(this, false);
       	 //	缓存到 attached scrap
         mAttachedScrap.add(holder);                                                         
     } else {
         if (mChangedScrap == null) {                                                        
             mChangedScrap = new ArrayList<ViewHolder>();                                    
         }                                                                                   
         holder.setScrapContainer(this, true);
       	 // 缓存到 changed scrap
         mChangedScrap.add(holder);                                                          
     }                                                                                       
 }                                                                                           
```





