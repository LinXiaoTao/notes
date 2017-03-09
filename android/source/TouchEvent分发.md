ACTION_DOWN或ACTION_POINTER_DOWN和ACTION_HOVER_MOVE 这三个事件称为**初始事件**，因为只有在分发这三个事件时候，才会处理触摸目标。

ACTION_POINTER_DOWN和ACTION_HOVER_MOVE这两个称为**多指初始事件**。

触摸目标指消费了**初始事件**的子视图，数据结构为链表。新增加的子视图为存放在头部。

viewFlags & CLICKABLE == CLICKABLE || viewFlags & LONG_CLICKABLE == LONG_CLICKABLE || viewFlags & CONTEXT_CLICKABLE == CONTEXT_CLICKABLE 表示该视图是 "可点击的"。

1. Android的Touch事件由Activity->ViewGroup-View按顺序分发(这里只关心到Activity)
2. 父视图可以通过`onInterceptTouchEvent`来拦截事件分发到子视图
3. 子视图可以调用`requestDisallowInterceptTouchEvent`来禁止父视图拦截事件，但在ACTION_DOWN时，会清除该标记，即只在ACTION_DOWN时，调用该方法没有效果。
4. 如果子视图没有调用`requestDisallowInterceptTouchEvent`，那么父视图任何时候都可以拦截事件，即使子视图已经消费了ACTION_DOWN，拦截事件后，会向子视图分发一个ACTION_CANCEL事件
5. 父视图分发事件时，只会在**初始事件**时候，来设置**触摸目标**。即如果子视图没消费这三个事件，也不会接收到后续的事件。
6. 当分发**多指初始事件**时，没有新的子视图来消费该事件时，会将该事件的指针赋予**触摸目标**的尾部。
7. 当没有子视图来消费事件时，会将事件分发给父视图。(super.dispatchTouchEvent)
8. View的事件分发，OnTouchListener.onTouch -> onTouchEvent -> TouchDelegate.onTouchEvent -> onClick/onLongClick等等
9. 只要视图是**可点击的**就默认会消费事件，即使该视图是**不启用的(setEnabled(false))**，只是不做点击处理反应。



Activity中

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            //用户交互回调
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
           //window是否消费该事件,touch事件分发从这里开始
            return true;
        }
  		//当前view没有消费事件，则由activity处理
        return onTouchEvent(ev);
    }

//当前没有任何view去消费事件时
public boolean onTouchEvent(MotionEvent event) {
  		//window是否消费该事件，来关闭当前window
        if (mWindow.shouldCloseOnTouch(this, event)) {
            finish();
            return true;
        }

        return false;
    }
```

PhoneWindow中

```Java
@Override
  public boolean superDispatchTouchEvent(MotionEvent event) {
        //window将事件委托给DecorView分发
        return mDecor.superDispatchTouchEvent(event);
    }
```

DecorView中

```java
 public boolean superDispatchTouchEvent(MotionEvent event) {
   		//调用父类的dispatchTouchEvent进行分发
        return super.dispatchTouchEvent(event);
    }
```

ViewGroup中

```java
@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
      	//输入事件一致性验证者
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
        }

        // If the event targets the accessibility focused view and this is it, start
        // normal event dispatch. Maybe a descendant is what will handle the click.
      //辅助功能处理
        if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
            ev.setTargetAccessibilityFocus(false);
        }
		//从这里开始事件分发
        boolean handled = false;
        //安全策略过滤
        if (onFilterTouchEventForSecurity(ev)) {
            final int action = ev.getAction();
            //多指处理
          	//多指情况下，POINTER:指针。(可以理解为单个手指)
            final int actionMasked = action & MotionEvent.ACTION_MASK;

            // Handle an initial down.
          	//处理一个初始按下事件ACTION_DOWN
            if (actionMasked == MotionEvent.ACTION_DOWN) {
              //丢弃之前的状态
                // Throw away all previous state when starting a new touch gesture.
                // The framework may have dropped the up or cancel event for the previous gesture
                // due to an app switch, ANR, or some other state change.
              //取消，清除所有的触摸目标
                cancelAndClearTouchTargets(ev);
              //重置触摸状态，这里会将使用requestDisallowInterceptTouchEvent设置的状态清除
              //这个状态可以阻止父视图(通过onInterceptTouchEvent)拦截子视图Touch事件
                resetTouchState();
            }

            // Check for interception.
          	//检查是否拦截
            final boolean intercepted;
          	//当为ACTION_DEON事件或触摸目标为不为null
          	//即使已经有子视图消费了事件，依然会执行判断
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
              	//检查子视图是否调用了requestDisallowInterceptTouchEvent来禁止父视图拦截
              	//这个标记为ACTION_DOWN处理中被清除，即子视图在ACTION_DOWN中调用是不起作用
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                  	//检查当前父视图是否拦截该事件
                    intercepted = onInterceptTouchEvent(ev);
                  	//恢复action，以防止在onInterceptTouchEvent被改变
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
              	//当前不是ACTION_DOWN事件，且没有触摸目标
              	//即子视图如果不消费ACTION_DOWN，那么后续事件也不会分发到。
                intercepted = true;
            }

            // If intercepted, start normal event dispatch. Also if there is already
            // a view that is handling the gesture, do normal event dispatch.
          	//辅助功能
            if (intercepted || mFirstTouchTarget != null) {
                ev.setTargetAccessibilityFocus(false);
            }

            // Check for cancelation.
          	//检查是否为ACTION_CANCEL事件
            final boolean canceled = resetCancelNextUpFlag(this)
                    || actionMasked == MotionEvent.ACTION_CANCEL;

            // Update list of touch targets for pointer down, if needed.
          	//检查是否把事件分发给多个子视图
          	//通过setMotionEventSplittingEnabled设置，Android3.0以上默认开启
            final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
            TouchTarget newTouchTarget = null;
            boolean alreadyDispatchedToNewTouchTarget = false;
          	//当事件不取消而且不拦截
            if (!canceled && !intercepted) {

                // If the event is targeting accessiiblity focus we give it to the
                // view that has accessibility focus and if it does not handle it
                // we clear the flag and dispatch the event to all children as usual.
                // We are looking up the accessibility focused host to avoid keeping
                // state since these events are very rare.
              	//辅助功能
                View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                        ? findChildWithAccessibilityFocus() : null;
				//1.当ACTION_DOWN事件(第一次按下)
              	//2.或事件可以分发给多个子视图且ACTION+POINTER_DOWN(多指处理，当主指针还没结束，又按下一个指针)
              	//3.或多指情况下，指针移动
              	//这里的判断表示，只有在ACTION_DOWN或者当前事件分发还没结束，出现新的指针事件才会进行分发。但这里的前提依然是，事件未取消和父视图不拦截，和至少接收了ACTION_DOWN事件(即触摸目标链表不为空，ACTION_DOWN事件除外)
                if (actionMasked == MotionEvent.ACTION_DOWN
                        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                  	//获取事件的指针索引
                    final int actionIndex = ev.getActionIndex(); // always 0 for down
                  	//指针id的位掩码(用于获取触摸目标捕获的指针)
                  	//TouchTarget.pointerIdBis 指针id的组合位掩码，用于目标捕获的所有指针
                    final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                            : TouchTarget.ALL_POINTER_IDS;

                    // Clean up earlier touch targets for this pointer id in case they
                    // have become out of sync.
                  	//清除这个指针作用的早先的触摸目标
                    removePointersFromTouchTargets(idBitsToAssign);

                    final int childrenCount = mChildrenCount;
                  	//扫描可以接收事件的子视图
                    if (newTouchTarget == null && childrenCount != 0) {
                        final float x = ev.getX(actionIndex);
                        final float y = ev.getY(actionIndex);
                        // Find a child that can receive the event.
                        // Scan children from front to back.
                      	//获取子视图的事件分发列表
                      	//这里考虑了自定义的子视图绘制顺序
                        final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                      //TODO:按buildTouchDispatchChildList()的返回 != null，即customOrder永远为false，即自定义了绘制顺序也不会改变事件分发顺序
                      final boolean customOrder = preorderedList == null
                                && isChildrenDrawingOrderEnabled();
                        final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);

                            // If there is a view that has accessibility focus we want it
                            // to get the event first and if not handled we will perform a
                            // normal dispatch. We may do a double iteration but this is
                            // safer given the timeframe.
                          	//辅助功能
                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }
							
                          	//检查子视图是否可以接受事件和触摸事件是否作用在子视图上
                            if (!canViewReceivePointerEvents(child)
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }
							
                          	//检查当前子视图是否已经存在触摸目标链表中
                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                              	//如果已经存在，则将新指针id赋值给它，并退出循环
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }
							//检查是否设置了取消标记
                          	//TODO:但这里没对结果进行处理
                            resetCancelNextUpFlag(child);
                          	//在这里将事件交予子视图进行分发
                          	//一般能执行到这里的:
                          	//1.ACTION_DOWN
                          	//2.多指情况下，指针作用于"新"视图上
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                              	//如果子视图消费了事件
                                // Child wants to receive touch within its bounds.
                              	//保存一些值
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex points into presorted list, find original index
                                    for (int j = 0; j < childrenCount; j++) {
                                        if (children[childIndex] == mChildren[j]) {
                                            mLastTouchDownIndex = j;
                                            break;
                                        }
                                    }
                                } else {
                                    mLastTouchDownIndex = childIndex;
                                }
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                              	//设置接收事件的子视图为新的触摸目标，并设置为触摸目标链表的头
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }

                            // The accessibility focus didn't handle the event, so clear
                            // the flag and do a normal dispatch to all children.
                            ev.setTargetAccessibilityFocus(false);
                        }
                        if (preorderedList != null) preorderedList.clear();
                    }

                    if (newTouchTarget == null && mFirstTouchTarget != null) {
                        // Did not find a child to receive the event.
                        // Assign the pointer to the least recently added target.
                      	//当触摸目标链表不为空，且当前事件没有产生新的触摸目标(包含已有的触摸目标)来接收
                        newTouchTarget = mFirstTouchTarget;
                        while (newTouchTarget.next != null) {
                            newTouchTarget = newTouchTarget.next;
                        }
                      	//找到触摸目标链表的尾部，将指针作用于它
                        newTouchTarget.pointerIdBits |= idBitsToAssign;
                    }
                }
            }

            // Dispatch to touch targets.
          	//除了ACTION_DOWN，ACTION_POINTER_DOWN,ACTION_HOVER_MOVE
          	//其他的事件的分发都从这里
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
              	//没有子视图接收事件，将事件分发给当前视图
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
                // Dispatch to touch targets, excluding the new touch target if we already
                // dispatched to it.  Cancel touch targets if necessary.
              	//将事件分发给触摸目标，除了新的触摸目标，因为在之前已经分发了
                TouchTarget predecessor = null;
                TouchTarget target = mFirstTouchTarget;
                while (target != null) {
                    final TouchTarget next = target.next;
                    if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                      	//新的触摸目标，直接跳过
                        handled = true;
                    } else {
                      	//检查事件是否被取消或者被拦截
                      	//即使子视图在ACTION_DOWN中接收了事件，如果没禁止拦截，父视图依然可以拦截事件
                        final boolean cancelChild = resetCancelNextUpFlag(target.child)
                                || intercepted;
                      	//分发到触摸目标
                      	//当cancelChild=true,会分发ACTION_CANCEL事件
                      	//dispatchTransformedTouchEvent中会判断当前触摸目标是否包含指针id
                        if (dispatchTransformedTouchEvent(ev, cancelChild,
                                target.child, target.pointerIdBits)) {
                            handled = true;
                        }
                      	//当事件被取消，清除触摸目标链表
                        if (cancelChild) {
                            if (predecessor == null) {
                                mFirstTouchTarget = next;
                            } else {
                                predecessor.next = next;
                            }
                            target.recycle();
                            target = next;
                            continue;
                        }
                    }
                    predecessor = target;
                    target = next;
                }
            }

            // Update list of touch targets for pointer up or cancel, if needed.
            if (canceled
                    || actionMasked == MotionEvent.ACTION_UP
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                resetTouchState();
            } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
                final int actionIndex = ev.getActionIndex();
                final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
                removePointersFromTouchTargets(idBitsToRemove);
            }
        }

        if (!handled && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
        }
        return handled;
    }
```

````java
//取消和清除所有触摸目标
private void cancelAndClearTouchTargets(MotionEvent event) {
        if (mFirstTouchTarget != null) {
            boolean syntheticEvent = false;
            if (event == null) {
                final long now = SystemClock.uptimeMillis();
                //获取一个ACTION_CANCEL事件
                event = MotionEvent.obtain(now, now,
                        MotionEvent.ACTION_CANCEL, 0.0f, 0.0f, 0);
                event.setSource(InputDevice.SOURCE_TOUCHSCREEN);
                syntheticEvent = true;
            }

            for (TouchTarget target = mFirstTouchTarget; target != null; target = target.next) {
                resetCancelNextUpFlag(target.child);
               //分发ACTION_CANCEL事件
                dispatchTransformedTouchEvent(event, true, target.child, target.pointerIdBits);
            }
            clearTouchTargets();

            if (syntheticEvent) {
                event.recycle();
            }
        }
    }
//清除所有触摸目标
 private void clearTouchTargets() {
        TouchTarget target = mFirstTouchTarget;
        if (target != null) {
            do {
                TouchTarget next = target.next;
                target.recycle();
                target = next;
            } while (target != null);
            mFirstTouchTarget = null;
        }
    }
````

```java
//将事件分发到相应的目标
private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
        final boolean handled;

        // Canceling motions is a special case.  We don't need to perform any transformations
        // or filtering.  The important part is the action, not the contents.
  		//当事件取消时，会根据是否有触摸目标，向当前视图或子视图分发一个ACTION_CANCEL事件
        final int oldAction = event.getAction();
        if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
            event.setAction(MotionEvent.ACTION_CANCEL);
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
            }
            event.setAction(oldAction);
            return handled;
        }

        // Calculate the number of pointers to deliver.
  		//计算要传递的指针
        final int oldPointerIdBits = event.getPointerIdBits();
        final int newPointerIdBits = oldPointerIdBits & desiredPointerIdBits;

        // If for some reason we ended up in an inconsistent state where it looks like we
        // might produce a motion event with no pointers in it, then drop the event.
  		//该触摸目标不包含该指针,不分发
        if (newPointerIdBits == 0) {
            return false;
        }

        // If the number of pointers is the same and we don't need to perform any fancy
        // irreversible transformations, then we can reuse the motion event for this
        // dispatch as long as we are careful to revert any changes we make.
        // Otherwise we need to make a copy.
  		//将事件进行分发，根据是否有触摸目标
        final MotionEvent transformedEvent;
        if (newPointerIdBits == oldPointerIdBits) {
            if (child == null || child.hasIdentityMatrix()) {
                if (child == null) {
                    handled = super.dispatchTouchEvent(event);
                } else {
                    final float offsetX = mScrollX - child.mLeft;
                    final float offsetY = mScrollY - child.mTop;
                    event.offsetLocation(offsetX, offsetY);

                    handled = child.dispatchTouchEvent(event);

                    event.offsetLocation(-offsetX, -offsetY);
                }
                return handled;
            }
            transformedEvent = MotionEvent.obtain(event);
        } else {
            transformedEvent = event.split(newPointerIdBits);
        }

        // Perform any necessary transformations and dispatch.
        if (child == null) {
            handled = super.dispatchTouchEvent(transformedEvent);
        } else {
            final float offsetX = mScrollX - child.mLeft;
            final float offsetY = mScrollY - child.mTop;
            transformedEvent.offsetLocation(offsetX, offsetY);
            if (! child.hasIdentityMatrix()) {
                transformedEvent.transform(child.getInverseMatrix());
            }

            handled = child.dispatchTouchEvent(transformedEvent);
        }

        // Done.
        transformedEvent.recycle();
        return handled;
    }
```

View中

```java
public boolean dispatchTouchEvent(MotionEvent event) {
  		//辅助功能
        // If the event should be handled by accessibility focus first.
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.
            event.setTargetAccessibilityFocus(false);
        }

        boolean result = false;
		
  		//调试作用
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
          	//停止嵌套滚动
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
          	//安全策略过滤
          	//处理拖动滚动条
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
          	//先调用OnTouchListener回调
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }
			
          	//如果没有OnTouchListener或者没有消费，则调用onTouchEvent
            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }
	
  		//调试
        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
  		//当ACTION_UP，ACTION_CANCEL或ACTION_DOWN且不消费，停止嵌套滚动
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }
```

```java
public boolean onTouchEvent(MotionEvent event) {
        final float x = event.getX();
        final float y = event.getY();
        final int viewFlags = mViewFlags;
        final int action = event.getAction();

        if ((viewFlags & ENABLED_MASK) == DISABLED) {
          	//当该视图不启用时(setEnabled(false))
            if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                setPressed(false);
            }
            // A disabled view that is clickable still consumes the touch
            // events, it just doesn't respond to them.
          	//不启用的视图依然可以消费事件，只是没有反应而已。
          	//只要该视图是 "可点击的"
            return (((viewFlags & CLICKABLE) == CLICKABLE
                    || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                    || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE);
        }
  		//调用TouchDelegate
        if (mTouchDelegate != null) {
            if (mTouchDelegate.onTouchEvent(event)) {
                return true;
            }
        }
		
        if (((viewFlags & CLICKABLE) == CLICKABLE ||
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) ||
                (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE) {
          	//该视图是 "可点击的"
            switch (action) {
                case MotionEvent.ACTION_UP:
                    boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                    if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                      	//当我们设置 "按下" 状态或 "延时按下" 状态
                        // take focus if we don't have it already and we should in
                        // touch mode.
                      	//获取焦点
                        boolean focusTaken = false;
                        if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                            focusTaken = requestFocus();
                        }

                        if (prepressed) {
                            // The button is being released before we actually
                            // showed it as pressed.  Make it show the pressed
                            // state now (before scheduling the click) to ensure
                            // the user sees it.
                          	//如果设置了 "延时按下" 状态，那在实际释放之前设置为 "按下" 状态
                            setPressed(true, x, y);
                       }

                        if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                          	//如果没有调用长按动作而且不忽略下一个抬起的事件，清除长按回调，不回调长按事件
                            // This is a tap, so remove the longpress check
                            removeLongPressCallback();

                            // Only perform take click actions if we were in the pressed state
                            if (!focusTaken) {
                              	//获取不了焦点，表示正在按下状态，才处理点击事件
                                // Use a Runnable and post this rather than calling
                                // performClick directly. This lets other visual state
                                // of the view update before click actions start.
                                if (mPerformClick == null) {
                                    mPerformClick = new PerformClick();
                                }
                                if (!post(mPerformClick)) {
                                    performClick();
                                }
                            }
                        }

                        if (mUnsetPressedState == null) {
                            mUnsetPressedState = new UnsetPressedState();
                        }
						
                      	//清除 "按下" 状态
                        if (prepressed) {
                            postDelayed(mUnsetPressedState,
                                    ViewConfiguration.getPressedStateDuration());
                        } else if (!post(mUnsetPressedState)) {
                            // If the post failed, unpress right now
                            mUnsetPressedState.run();
                        }
						
                       //取消点击检测计时器。即 "延时按下" 状态
                        removeTapCallback();
                    }
                    mIgnoreNextUpEvent = false;
                    break;

                case MotionEvent.ACTION_DOWN:
                    mHasPerformedLongPress = false;
					
                	//执行按钮的相关操作
                	//比如菜单栏点击
                    if (performButtonActionOnTouchDown(event)) {
                        break;
                    }

                    // Walk up the hierarchy to determine if we're inside a scrolling container.
                	//检查当前是否位于滚动容器中
                    boolean isInScrollingContainer = isInScrollingContainer();

                    // For views inside a scrolling container, delay the pressed feedback for
                    // a short period in case this is a scroll.
                	//当父视图处于滚动中，则延时处理 "按下" 状态
                    if (isInScrollingContainer) {
                        mPrivateFlags |= PFLAG_PREPRESSED;
                        if (mPendingCheckForTap == null) {
                            mPendingCheckForTap = new CheckForTap();
                        }
                        mPendingCheckForTap.x = event.getX();
                        mPendingCheckForTap.y = event.getY();
                        postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                    } else {
                        // Not inside a scrolling container, so show the feedback right away
                        setPressed(true, x, y);
                        checkForLongClick(0, x, y);
                    }
                    break;

                case MotionEvent.ACTION_CANCEL:
                	//当事件取消时候，清理
                    setPressed(false);
                    removeTapCallback();
                    removeLongPressCallback();
                    mInContextButtonPress = false;
                    mHasPerformedLongPress = false;
                    mIgnoreNextUpEvent = false;
                    break;

                case MotionEvent.ACTION_MOVE:
                    drawableHotspotChanged(x, y);

                    // Be lenient about moving outside of buttons
                	//当手指移动时候，不再处于当前视图范围，取消事件
                    if (!pointInView(x, y, mTouchSlop)) {
                        // Outside button
                        removeTapCallback();
                        if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                            // Remove any future long press/tap checks
                            removeLongPressCallback();

                            setPressed(false);
                        }
                    }
                    break;
            }
			
          	//不管有没有处理，只要该视图具有 "可点击" ，就会默认消费掉该事件
         	//即使该视图不启用，依然会消费事件，只是不执行 "点击" 事件。
            return true;
        }

        return false;
    }
```

