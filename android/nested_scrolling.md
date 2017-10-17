## 概述

NestedScrolling 机制工作的流程如下：

1. 当 child view 开启滚动时，先通知 parent view 消费滚动事件
2. child view 消费剩余滚动事件
3. child view 通知 parent view 剩余的滚动事件



### 相关类

* `NestedScrollingChild`
* `NestedScrollingParent`
* `NestedScrollingChildHelper`
* `NestedScrollingParentHelper`

要实现 NestedScrolling 机制，parent view 需要实现 `NestedScrollingParent` 接口，child view 则需要实现 `NestedScrollingChild` 接口。



#### NestedScrollingChild

``` java
//开始嵌套滚动，返回 true 表示存在 parent view 进行嵌套滚动
public boolean startNestedScroll(int axes);
//结束嵌套滚动
public void stopNestedScroll();

//child view 处理之前，预先分发到 parent view 进行消费，返回 true 表示部分消费或者全部消费
public boolean dispatchNestedPreScroll(int dx, int dy, int[] consumed, int[] offsetInWindow);
//child view 处理后，将剩下的事件分发到 parent view，返回 true 表示分发成功
public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, int[] offsetInWindow);

//child view 处理之前，预先分发到 parent view 进行消费，返回 true，表示 parent view 消费了这次 fling
public boolean dispatchNestedPreFling(float velocityX, float velocityY);
//child view 处理后，将处理结果分发到 parent view，consumed 为 true，表示 child view 已消费
//返回 true 表示 parent view 消费了或者已其他方式处理
public boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed);
```



#### NestedScrollingParent

``` java
//响应 child view 的 startNestedScroll，返回 true，表示进行嵌套滚动
public boolean onStartNestedScroll(View child, View target, int nestedScrollAxes);
//成功获取嵌套滚动处理，在 onStartNestedScroll 返回 true 之后调用
public void onNestedScrollAccepted(View child, View target, int nestedScrollAxes);
//停止嵌套滚动时调用
public void onStopNestedScroll(View target);

//dispatchNestedPreScroll 之后调用
public void onNestedPreScroll(View target, int dx, int dy, int[] consumed);
//dispatchNestedScroll 之后调用
public void onNestedScroll(View target, int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed);

//dispatchNestedPreFling 之后调用
public boolean onNestedPreFling(View target, float velocityX, float velocityY);
//dispatchNestedFling 之后调用
public boolean onNestedFling(View target, float velocityX, float velocityY, boolean consumed);
```



### 源码分析

基于 Android-25，android.view.View

``` java
public boolean startNestedScroll(int axes) {
        if (hasNestedScrollingParent()) {
            // Already in progress
          	//存在嵌套滚动的 parent view，直接返回 true
            return true;
        }
        if (isNestedScrollingEnabled()) {
          	//启用了嵌套滚动
          	//遍历 parent view
            ViewParent p = getParent();
            View child = this;
            while (p != null) {
                try {
                    if (p.onStartNestedScroll(child, this, axes)) {
                      	//onStartNestedScroll 返回 true
                        mNestedScrollingParent = p;
                      	//随后调用 onNestedScrollAccepted
                        p.onNestedScrollAccepted(child, this, axes);
                        return true;
                    }
                } catch (AbstractMethodError e) {
                    Log.e(VIEW_LOG_TAG, "ViewParent " + p + " does not implement interface " +
                            "method onStartNestedScroll", e);
                    // Allow the search upward to continue
                }
                if (p instanceof View) {
                    child = (View) p;
                }
                p = p.getParent();
            }
        }
        return false;
    }
```

``` java
public boolean dispatchNestedPreScroll(int dx, int dy,
            @Nullable @Size(2) int[] consumed, @Nullable @Size(2) int[] offsetInWindow) {
        if (isNestedScrollingEnabled() && mNestedScrollingParent != null) {
            if (dx != 0 || dy != 0) {
                int startX = 0;
                int startY = 0;
                if (offsetInWindow != null) {
                  	//保存当前位置
                    getLocationInWindow(offsetInWindow);
                    startX = offsetInWindow[0];
                    startY = offsetInWindow[1];
                }

                if (consumed == null) {
                    if (mTempNestedScrollConsumed == null) {
                        mTempNestedScrollConsumed = new int[2];
                    }
                    consumed = mTempNestedScrollConsumed;
                }
                consumed[0] = 0;
                consumed[1] = 0;
              	//调用 onNestedPreScroll
                mNestedScrollingParent.onNestedPreScroll(this, dx, dy, consumed);

                if (offsetInWindow != null) {
                  	//offsetInWindow 保存偏移量
                    getLocationInWindow(offsetInWindow);
                    offsetInWindow[0] -= startX;
                    offsetInWindow[1] -= startY;
                }
              	//返回是否消费事件
                return consumed[0] != 0 || consumed[1] != 0;
            } else if (offsetInWindow != null) {
                offsetInWindow[0] = 0;
                offsetInWindow[1] = 0;
            }
        }
        return false;
    }
```

``` java
public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, @Nullable @Size(2) int[] offsetInWindow) {
        if (isNestedScrollingEnabled() && mNestedScrollingParent != null) {
            if (dxConsumed != 0 || dyConsumed != 0 || dxUnconsumed != 0 || dyUnconsumed != 0) {
                int startX = 0;
                int startY = 0;
                if (offsetInWindow != null) {
                    getLocationInWindow(offsetInWindow);
                    startX = offsetInWindow[0];
                    startY = offsetInWindow[1];
                }
				//调用 onNestedScroll
                mNestedScrollingParent.onNestedScroll(this, dxConsumed, dyConsumed,
                        dxUnconsumed, dyUnconsumed);

                if (offsetInWindow != null) {
                    getLocationInWindow(offsetInWindow);
                    offsetInWindow[0] -= startX;
                    offsetInWindow[1] -= startY;
                }
              	//返回是否分发成功
                return true;
            } else if (offsetInWindow != null) {
                // No motion, no dispatch. Keep offsetInWindow up to date.
                offsetInWindow[0] = 0;
                offsetInWindow[1] = 0;
            }
        }
        return false;
    }
```

``` java
public boolean dispatchNestedPreFling(float velocityX, float velocityY) {
        if (isNestedScrollingEnabled() && mNestedScrollingParent != null) {
          	//调用 onNestedPreFling
            return mNestedScrollingParent.onNestedPreFling(this, velocityX, velocityY);
        }
        return false;
    }
```

``` java
 public boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed) {
        if (isNestedScrollingEnabled() && mNestedScrollingParent != null) {
          	//调用 onNestedFling
            return mNestedScrollingParent.onNestedFling(this, velocityX, velocityY, consumed);
        }
        return false;
    }
```

``` java
public void stopNestedScroll() {
        if (mNestedScrollingParent != null) {
          	//调用 onStopNestedScroll
            mNestedScrollingParent.onStopNestedScroll(this);
            mNestedScrollingParent = null;
        }
    }
```