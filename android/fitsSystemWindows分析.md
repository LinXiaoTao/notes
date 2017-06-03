ViewRootImpl

``` java
private void performTraversals(){
  //...
   dispatchApplyInsets(host);
  //...
} 

void dispatchApplyInsets(View host) {
   		//host 为 DecorView
        host.dispatchApplyWindowInsets(getWindowInsets(true /* forceConstruct */));
    }
```

ViewGroup

``` java
@Override
    public WindowInsets dispatchApplyWindowInsets(WindowInsets insets) {
      	//先调用超类 View.dispatchApplyWindowInsets(WindowInsets)
        insets = super.dispatchApplyWindowInsets(insets);
        if (!insets.isConsumed()) {
          	//如果没被消费
            final int count = getChildCount();
            for (int i = 0; i < count; i++) {
              	//传递给子视图
                insets = getChildAt(i).dispatchApplyWindowInsets(insets);
              	//某个子视图消费后则中止传递
                if (insets.isConsumed()) {
                    break;
                }
            }
        }
        return insets;
    }
```

View

``` java
public WindowInsets dispatchApplyWindowInsets(WindowInsets insets) {
        try {
            mPrivateFlags3 |= PFLAG3_APPLYING_INSETS;
            if (mListenerInfo != null && mListenerInfo.mOnApplyWindowInsetsListener != null) {
              //可以通过 OnApplyWindowInsetsListener 来处理
                return mListenerInfo.mOnApplyWindowInsetsListener.onApplyWindowInsets(this, insets);
            } else {
              //默认情况下
                return onApplyWindowInsets(insets);
            }
        } finally {
            mPrivateFlags3 &= ~PFLAG3_APPLYING_INSETS;
        }
    }


public WindowInsets onApplyWindowInsets(WindowInsets insets) {
        if ((mPrivateFlags3 & PFLAG3_FITTING_SYSTEM_WINDOWS) == 0) {
            // We weren't called from within a direct call to fitSystemWindows,
            // call into it as a fallback in case we're in a class that overrides it
            // and has logic to perform.
          	//调用 fitSystemWindows(Rect)
          	//第一次没有设置 PFLAG3_FITTING_SYSTEM_WINDOWS
          	//兼容旧方法 fitSystemWindows
            if (fitSystemWindows(insets.getSystemWindowInsets())) {
                return insets.consumeSystemWindowInsets();
            }
        } else {
            // We were called from within a direct call to fitSystemWindows.
            if (fitSystemWindowsInt(insets.getSystemWindowInsets())) {
                return insets.consumeSystemWindowInsets();
            }
        }
        return insets;
    }

protected boolean fitSystemWindows(Rect insets) {
        if ((mPrivateFlags3 & PFLAG3_APPLYING_INSETS) == 0) {
          	//还没有设置 PFLAG3_APPLYING_INSETS
            if (insets == null) {
                // Null insets by definition have already been consumed.
                // This call cannot apply insets since there are none to apply,
                // so return false.
                return false;
            }
            // If we're not in the process of dispatching the newer apply insets call,
            // that means we're not in the compatibility path. Dispatch into the newer
            // apply insets path and take things from there.
            try {
                mPrivateFlags3 |= PFLAG3_FITTING_SYSTEM_WINDOWS;
              	//设置 PFLAG3_FITTING_SYSTEM_WINDOWS，继续调用 dispatchApplyWindowInsets
                return dispatchApplyWindowInsets(new WindowInsets(insets)).isConsumed();
            } finally {
                mPrivateFlags3 &= ~PFLAG3_FITTING_SYSTEM_WINDOWS;
            }
        } else {
            // We're being called from the newer apply insets path.
            // Perform the standard fallback behavior.
          	//处理 fitSystemWindows
            return fitSystemWindowsInt(insets);
        }
    }


private boolean fitSystemWindowsInt(Rect insets) {
        if ((mViewFlags & FITS_SYSTEM_WINDOWS) == FITS_SYSTEM_WINDOWS) {
          	//fitSystemWindows = true 控件将调整它的 padding 来适应系统 windows 比如 状态栏
            mUserPaddingStart = UNDEFINED_PADDING;
            mUserPaddingEnd = UNDEFINED_PADDING;
            Rect localInsets = sThreadLocal.get();
            if (localInsets == null) {
                localInsets = new Rect();
                sThreadLocal.set(localInsets);
            }
            boolean res = computeFitSystemWindows(insets, localInsets);
            mUserPaddingLeftInitial = localInsets.left;
            mUserPaddingRightInitial = localInsets.right;
            internalSetPadding(localInsets.left, localInsets.top,
                    localInsets.right, localInsets.bottom);
            return res;
        }
        return false;
    }
```

### 总结

当设置 **windowTranslucentStatus = true(设置 statusColor = transparent 不起作用) 或 FULLSCREEN等等** 一般来说，WindowInsets 的分发从 `ViewGroup.dispatchApplyWindowInsets(WindowInsets)` 开始，首先调用超类 View 的 `dispatchApplyWindowInsets(WindowInsets)` 来消费 WindowInsets，如果还消费完，则向子视图分发，其中某个子视图消费完了，则中止分发。在 View 的 `dispatchApplyWindowInsets(WindowInsets)` 则判断是否设置了 `OnApplyWindowInsetsListener` ，交由其处理，否则调用 `onApplyWindowInsets(WindowInsets)`进行消费，其中会对旧方法 `fitSystemWindows` 进行兼容，最终会调用 `fitSystemWindowsInt(Rect)` 来通过设置 padding，来进行调整。