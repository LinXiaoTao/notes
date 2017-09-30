## Touch 事件分发

`ACTION_DOWN` 或 `ACTION_POINTER_DOWN` 和 `ACTION_HOVER_MOVE` 这三个事件称为**初始事件**，因为只有在分发这三个事件时候，才会处理触摸目标。

`ACTION_POINTER_DOWN` 和 `ACTION_HOVER_MOVE` 这两个称为**多指初始事件**。

触摸目标指消费了**初始事件**的子视图，数据结构为链表。新增加的子视图为存放在头部。



### 结论

1. Android 的 Touch 事件由 **Activity -> ViewGroup -> View** 按顺序分发（这里只关心到 Activity）

   ​

2. 子视图可以调用 `requestDisallowInterceptTouchEvent(boolean)`  来禁止父视图拦截事件

   ``` java
   //在 ViewGroup.dispatchTouchEvent(MotionEvent) 中，会清除掉这个标记
   if (actionMasked == MotionEvent.ACTION_DOWN) {
     cancelAndClearTouchTargets(ev);
     //清除标记
     resetTouchState();
   }
   ```

   ​

3. 父视图可以通过 `onInterceptTouchEvent(MotionEvent)` 来拦截事件分发到子视图。

   如果子视图没有调用 `requestDisallowInterceptTouchEvent(boolean)`，那么父视图任何时候都可以拦截事件，即使子视图已经消费了 `ACTION_DOWN`，拦截事件后，会将该事件 `setAction(ACTION_CANCEL)`，分发给子视图，之后的事件都将分发给父视图，同时将触摸目标置为 null。

   ``` java
   //ViewGroup.dispatchTouchEvent(MotionEvent)
   final boolean intercepted;
   if (actionMasked == MotionEvent.ACTION_DOWN|| mFirstTouchTarget != null){
     //只要 ACTION_DOWN 或者 已经有产生事件消费，就会检测是否可拦截
     final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
     if (!disallowIntercept) {
     	intercepted = onInterceptTouchEvent(ev);
       ev.setAction(action);
     }eles{
       intercepted = false;
     }
   }else{
     intercepted = true;
   }

   //当事件被拦截后
   //ViewGroup.dispatchTouchEvent(MotionEvent)
   final boolean cancelChild = resetCancelNextUpFlag(target.child)|| intercepted;
   //cancelChild = true
   if (dispatchTransformedTouchEvent(ev, cancelChild,target.child, target.pointerIdBits)){
     handled = true;
   }
   if (cancelChild){
     //清除触摸目标
     if (predecessor == null){
        mFirstTouchTarget = next;
     }else{
       predecessor.next = next;
     }
      target.recycle();
      target = next;
      continue;
   }

   //ViewGroup.dispatchTouchEvent(MotionEvent)
   if(mFirstTouchTarget == null){
     //当前事件被拦截后，清除了触摸目标，剩下的事件都将分发到当前父视图
     handled = dispatchTransformedTouchEvent(ev, canceled, null,TouchTarget.ALL_POINTER_IDS);
   }

   //ViewGroup.dispatchTransformedTouchEvent(...)
   final int oldAction = event.getAction();
   //当前事件被拦截后，cancel = true
   if (cancel || oldAction == MotionEvent.ACTION_CANCEL){
     event.setAction(MotionEvent.ACTION_CANCEL);
     if (child == null){
       handled = super.dispatchTouchEvent(event);
     }else{
       handled = child.dispatchTouchEvent(event);
     }
     event.setAction(oldAction);
     return handled;
   }
   ```

   ​

4. 父视图分发事件时，只会在**初始事件**时候，来设置**触摸目标**。即如果子视图没消费这三个事件，也不会接收到后续的事件。

   一般情况下，只有在 `ACTION_DOWN `中，返回值起主要作用，它决定了你是否消费接下来的所有事件，在消费了 `ACTION_DOWN` 后，在其他 `action` 中，返回 false，依然可以接收到其他事件。

   ``` java
   //ViewGroup.dispatchTouchEvent(MotionEvent)
   if(actionMasked == MotionEvent.ACTION_DOWN || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN) || actionMasked == MotionEvent.ACTION_HOVER_MOVE){
     //......
     if(dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)){
       //......
       //添加新的触摸目标
        newTouchTarget = addTouchTarget(child, idBitsToAssign);
     }
   }
   ```

   ​

5. 当分发**多指初始事件**时，没有新的子视图来消费该事件时，会将该事件的指针赋予**触摸目标**的尾部。

   ``` java
   //ViewGroup.dispatchTouchEvent(MotionEvent)
   if (newTouchTarget == null && mFirstTouchTarget != null){
     newTouchTarget = mFirstTouchTarget;
     while (newTouchTarget.next != null){
       newTouchTarget = newTouchTarget.next;
     }
     newTouchTarget.pointerIdBits |= idBitsToAssign;
   }
   ```

   ​

6. 当没有子视图来消费事件时，会将事件分发给父视图。

   ``` java
   //ViewGroup.dispatchTouchEvent(MotionEvent)
   if(mFirstTouchTarget == null){
     handled = dispatchTransformedTouchEvent(ev, canceled, null,TouchTarget.ALL_POINTER_IDS);
   }
   ```

   ​

7. View的事件分发

   ``` java
   //View.dispatchTouchEvent(MotionEvent)
   ListenerInfo li = mListenerInfo;
   //1. onTouch 消费
   if(li != null && li.mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED && li.mOnTouchListener.onTouch(this, event)){
     result = true;
   }

   //View.dispatchTouchEvent(MotionEvent)
   //2. onTouchEvent 消费
   if (!result && onTouchEvent(event)){
      result = true;
   }
   //View.onTouchEvent(MotionEvent)
   //2.1 TouchDelegate 消费
   if (mTouchDelegate != null) {
     if (mTouchDelegate.onTouchEvent(event)){
       reture true;
     }
   }
   //View.onTouchEvent(MotionEvent)
   //2.2 onClick(View) 和 onLongClick(View) 消费

   ```

   ​

   ​

8. 只要视图是**可点击的**就默认会消费事件，即使该视图是 `setEnabled(false)`，只是不做点击处理反应。

   ``` java
   //View.onTouchEvent(MotionEvent)
   if(((viewFlags & CLICKABLE) == CLICKABLE || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE){
     //......
     reture true;
   }
   ```

   ​

   ​

