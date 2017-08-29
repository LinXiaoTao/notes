**本文基于 android-25**



`android.transition.TransitionManager`

``` java
public static void go(Scene scene, Transition transition) {
        changeScene(scene, transition);
    }
```

``` java
private static void changeScene(Scene scene, Transition transition) {

        final ViewGroup sceneRoot = scene.getSceneRoot();
        if (!sPendingTransitions.contains(sceneRoot)) {
          	//加入到等待 transition 的 view list
            sPendingTransitions.add(sceneRoot);

            Transition transitionClone = null;
            if (transition != null) {
                transitionClone = transition.clone();
                transitionClone.setSceneRoot(sceneRoot);
            }

            Scene oldScene = Scene.getCurrentScene(sceneRoot);
            if (oldScene != null && transitionClone != null &&
                    oldScene.isCreatedFromLayoutResource()) {
              	//isCreatedFromLayoutResource() == layoutID > 0
                transitionClone.setCanRemoveViews(true);
            }
			//预先设置
            sceneChangeSetup(sceneRoot, transitionClone);
			//调用 scene.enter()
            scene.enter();
			//执行 transition
            sceneChangeRunTransition(sceneRoot, transitionClone);
        }
    }
```

``` java
private static void sceneChangeSetup(ViewGroup sceneRoot, Transition transition) {

        // Capture current values
  		//获取当前运行的 transition
        ArrayList<Transition> runningTransitions = getRunningTransitions().get(sceneRoot);

        if (runningTransitions != null && runningTransitions.size() > 0) {
          	//暂停正在运行中的 transition == (anim.pause() && listeners.onTransitionPause())
            for (Transition runningTransition : runningTransitions) {
                runningTransition.pause(sceneRoot);
            }
        }

        if (transition != null) {
          	//transition 捕获所需起始 transiton value == transition.captureStartValues()
            transition.captureValues(sceneRoot, true);
        }

        // Notify previous scene that it is being exited
        Scene previousScene = Scene.getCurrentScene(sceneRoot);
        if (previousScene != null) {
          	//调用上一个 scene 退出
            previousScene.exit();
        }
    }
```

``` java
 private static void sceneChangeRunTransition(final ViewGroup sceneRoot,
            final Transition transition) {
        if (transition != null && sceneRoot != null) {
          	//MultiListener == (OnAttachStateChangeListener && OnPreDrawListener)
            MultiListener listener = new MultiListener(transition, sceneRoot);
          	//OnAttachStateChangeListener 可以在 onViewDetachedFromWindow 释放资源
            sceneRoot.addOnAttachStateChangeListener(listener);
          	//OnAttachStateChangeListener 可以再下一帧绘制之前执行 transition
            sceneRoot.getViewTreeObserver().addOnPreDrawListener(listener);
        }
    }
```



`android.transition.TransitionManager$MultiListener`

``` java
		@Override
        public void onViewDetachedFromWindow(View v) {
            removeListeners();
			// 从等待 transition 的 view list 中删除
            sPendingTransitions.remove(mSceneRoot);
            ArrayList<Transition> runningTransitions = getRunningTransitions().get(mSceneRoot);
            if (runningTransitions != null && runningTransitions.size() > 0) {
                for (Transition runningTransition : runningTransitions) {
                  //恢复 sceneChangeSetup() 中暂停的 transition
                    runningTransition.resume(mSceneRoot);
                }
            }
          	//清除使用的 transiton value
            mTransition.clearValues(true);
        }
```

``` java
		@Override
        public boolean onPreDraw() {
            removeListeners();

            // Don't start the transition if it's no longer pending.
            if (!sPendingTransitions.remove(mSceneRoot)) {
              	//没有添加到 等待 transition 的 view list，直接返回
              	//1.changeScene() 没有执行成功
              	//2.onViewDetachedFromWindow() 已经执行
                return true;
            }

            // Add to running list, handle end to remove it
            final ArrayMap<ViewGroup, ArrayList<Transition>> runningTransitions =
                    getRunningTransitions();
            ArrayList<Transition> currentTransitions = runningTransitions.get(mSceneRoot);
            ArrayList<Transition> previousRunningTransitions = null;
          	//保存之前正在运行中的 transiton list >> previousRunningTransitions
            if (currentTransitions == null) {
                currentTransitions = new ArrayList<Transition>();
                runningTransitions.put(mSceneRoot, currentTransitions);
            } else if (currentTransitions.size() > 0) {
                previousRunningTransitions = new ArrayList<Transition>(currentTransitions);
            }
          	//将需要执行的 transioton 添加到 运行中的 transition list
            currentTransitions.add(mTransition);
            mTransition.addListener(new Transition.TransitionListenerAdapter() {
                @Override
                public void onTransitionEnd(Transition transition) {
                  	//transiton end 删除执行的 transition
                    ArrayList<Transition> currentTransitions =
                            runningTransitions.get(mSceneRoot);
                    currentTransitions.remove(transition);
                }
            });
          	//在 sceneChangeSetup() 中捕获起始的 transiton value；在这里捕获结束的 transiton value == captureEndValues()
            mTransition.captureValues(mSceneRoot, false);
            if (previousRunningTransitions != null) {
                for (Transition runningTransition : previousRunningTransitions) {
                  	//恢复原先的 transition list
                    runningTransition.resume(mSceneRoot);
                }
            }
          	//开始执行当前 transition
            mTransition.playTransition(mSceneRoot);

            return true;
        }
```

> **sRunningTransitions 中已有的之前的 transition list 是怎么添加的？为什么一直存在**
>
> 答：sRunningTransitions 保存着所有 scene root 对应的 transitions。`onPreDraw()` 中添加当前 transition，`TransitionListener.onTransitionEnd()` 中删除。在执行 transition 时，如果还有 transition，就先暂停，等处理完当前 transition 再恢复。



`android.transition.Transition`

``` java
//TransitonManager 中在 sceneChangeSetup() 中调用和在 onPreDraw() 中调用，来捕获起始和结束属性值
void captureValues(ViewGroup sceneRoot, boolean start) {
        clearValues(start);
        if ((mTargetIds.size() > 0 || mTargets.size() > 0)
                && (mTargetNames == null || mTargetNames.isEmpty())
                && (mTargetTypes == null || mTargetTypes.isEmpty())) {
          	//优先处理设置了 TragetIds 和 TargetViews 的情况
          	//TragetIds 或 TargetViews 至少存在一个，并且 TargetNames 和 TargetTypes 不存在
            for (int i = 0; i < mTargetIds.size(); ++i) {
                int id = mTargetIds.get(i);
                View view = sceneRoot.findViewById(id);
                if (view != null) {
                  	//构造一个 TransitionValues
                    TransitionValues values = new TransitionValues();
                    values.view = view;
                    if (start) {
                      	//捕获起始 transiton value
                        captureStartValues(values);
                    } else {
                      	//捕获结束 transiton value
                        captureEndValues(values);
                    }
                    values.targetedTransitions.add(this);
                    capturePropagationValues(values);
                    if (start) {
                        addViewValues(mStartValues, view, values);
                    } else {
                        addViewValues(mEndValues, view, values);
                    }
                }
            }
          	//TragetViews 执行和 TargetIds 一样的逻辑。
            for (int i = 0; i < mTargets.size(); ++i) {
                View view = mTargets.get(i);
                TransitionValues values = new TransitionValues();
                values.view = view;
                if (start) {
                    captureStartValues(values);
                } else {
                    captureEndValues(values);
                }
                values.targetedTransitions.add(this);
                capturePropagationValues(values);
                if (start) {
                    addViewValues(mStartValues, view, values);
                } else {
                    addViewValues(mEndValues, view, values);
                }
            }
        } else {
          	//根据视图层次结构遍历捕获 transition value
            captureHierarchy(sceneRoot, start);
        }
        if (!start && mNameOverrides != null) {
          	//将使用 mNameOverrides 中的新的 transitonName 覆盖 mStartValues 的 transitionName
          	//mNameOverrides 中保存 mStartValues 中的 transitonName 和 新的 transitonName 中的映射
            int numOverrides = mNameOverrides.size();
            ArrayList<View> overriddenViews = new ArrayList<View>(numOverrides);
            for (int i = 0; i < numOverrides; i++) {
                String fromName = mNameOverrides.keyAt(i);
                overriddenViews.add(mStartValues.nameValues.remove(fromName));
            }
            for (int i = 0; i < numOverrides; i++) {
                View view = overriddenViews.get(i);
                if (view != null) {
                    String toName = mNameOverrides.valueAt(i);
                    mStartValues.nameValues.put(toName, view);
                }
            }
        }
    }
```
``` java
private void captureHierarchy(View view, boolean start) {
        if (view == null) {
            return;
        }
        int id = view.getId();
  		//过滤掉排除的视图
        if (mTargetIdExcludes != null && mTargetIdExcludes.contains(id)) {
            return;
        }
        if (mTargetExcludes != null && mTargetExcludes.contains(view)) {
            return;
        }
        if (mTargetTypeExcludes != null && view != null) {
            int numTypes = mTargetTypeExcludes.size();
            for (int i = 0; i < numTypes; ++i) {
                if (mTargetTypeExcludes.get(i).isInstance(view)) {
                    return;
                }
            }
        }
  		
        if (view.getParent() instanceof ViewGroup) {
          	//自身视图捕获
            TransitionValues values = new TransitionValues();
            values.view = view;
            if (start) {
                captureStartValues(values);
            } else {
                captureEndValues(values);
            }
            values.targetedTransitions.add(this);
            capturePropagationValues(values);
            if (start) {
                addViewValues(mStartValues, view, values);
            } else {
                addViewValues(mEndValues, view, values);
            }
        }
        if (view instanceof ViewGroup) {
            // Don't traverse child hierarchy if there are any child-excludes on this view				
          	//对子视图进行循环遍历捕获
          	//过滤排除的子视图
            if (mTargetIdChildExcludes != null && mTargetIdChildExcludes.contains(id)) {
                return;
            }
            if (mTargetChildExcludes != null && mTargetChildExcludes.contains(view)) {
                return;
            }
            if (mTargetTypeChildExcludes != null) {
                int numTypes = mTargetTypeChildExcludes.size();
                for (int i = 0; i < numTypes; ++i) {
                    if (mTargetTypeChildExcludes.get(i).isInstance(view)) {
                        return;
                    }
                }
            }
            ViewGroup parent = (ViewGroup) view;
            for (int i = 0; i < parent.getChildCount(); ++i) {
              	//子视图循环遍历
                captureHierarchy(parent.getChildAt(i), start);
            }
        }
    }
```

``` java
static void addViewValues(TransitionValuesMaps transitionValuesMaps,
            View view, TransitionValues transitionValues) {
  		//根据 view 保存 transition values
        transitionValuesMaps.viewValues.put(view, transitionValues);
  		//id 和 transiton name ，item id 都至多只能存一份，并且不能重复，如果存在重复，则不保存
  		//如果存在 3 个重复的，前两个会导致删除保存的，但第三个会重新添加进去，会有影响？
        int id = view.getId();
        if (id >= 0) {
            if (transitionValuesMaps.idValues.indexOfKey(id) >= 0) {
                // Duplicate IDs cannot match by ID.
                transitionValuesMaps.idValues.put(id, null);
            } else {
                transitionValuesMaps.idValues.put(id, view);
            }
        }
        String name = view.getTransitionName();
        if (name != null) {
            if (transitionValuesMaps.nameValues.containsKey(name)) {
                // Duplicate transitionNames: cannot match by transitionName.
                transitionValuesMaps.nameValues.put(name, null);
            } else {
                transitionValuesMaps.nameValues.put(name, view);
            }
        }
        if (view.getParent() instanceof ListView) {
            ListView listview = (ListView) view.getParent();
            if (listview.getAdapter().hasStableIds()) {
                int position = listview.getPositionForView(view);
                long itemId = listview.getItemIdAtPosition(position);
                if (transitionValuesMaps.itemIdValues.indexOfKey(itemId) >= 0) {
                    // Duplicate item IDs: cannot match by item ID.
                    View alreadyMatched = transitionValuesMaps.itemIdValues.get(itemId);
                    if (alreadyMatched != null) {
                        alreadyMatched.setHasTransientState(false);
                        transitionValuesMaps.itemIdValues.put(itemId, null);
                    }
                } else {
                    view.setHasTransientState(true);
                    transitionValuesMaps.itemIdValues.put(itemId, view);
                }
            }
        }
    }
```
> 捕获视图的属性值操作：
>
> 1. 如果存在 TargetIds 或 TragetViews，而且 TragetNames 和 TragetTypes 为空，则按 TargetIds 和 TragetViews 进行捕获
> 2. 如果不执行第 1 步的操作，则遍历 scene root 的所有子视图，对所有不排除的视图进行捕获。
>
> 捕获的属性值将通过 `addViewValues()` 分别存在 `mStartValues` 和 `mEndValues` 中。

```java
//判断是否为有效目标视图，用于 matchStartAndEnd() 
boolean isValidTarget(View target) {
    if (target == null) {
        return false;
    }
    int targetId = target.getId();
    //1.判断是否排除 ID
    if (mTargetIdExcludes != null && mTargetIdExcludes.contains(targetId)) {
        return false;
    }
    //2.判断是否为排除视图
    if (mTargetExcludes != null && mTargetExcludes.contains(target)) {
        return false;
    }
    //3.判断是否为排除类
    if (mTargetTypeExcludes != null && target != null) {
        int numTypes = mTargetTypeExcludes.size();
        for (int i = 0; i < numTypes; ++i) {
            Class type = mTargetTypeExcludes.get(i);
            if (type.isInstance(target)) {
                return false;
            }
        }
    }
    //4.是否为排除 transiton name
    if (mTargetNameExcludes != null && target != null && target.getTransitionName() != null) {
        if (mTargetNameExcludes.contains(target.getTransitionName())) {
            return false;
        }
    }
  	//5.没有目标过滤
    if (mTargetIds.size() == 0 && mTargets.size() == 0 &&
            (mTargetTypes == null || mTargetTypes.isEmpty()) &&
            (mTargetNames == null || mTargetNames.isEmpty())) {
        return true;
    }
  	//6.包含在目标 Id 和视图中
    if (mTargetIds.contains(targetId) || mTargets.contains(target)) {
        return true;
    }
  	//7.包含在目标 transition name 中
    if (mTargetNames != null && mTargetNames.contains(target.getTransitionName())) {
        return true;
    }
  	//8.包含在目标类中
    if (mTargetTypes != null) {
        for (int i = 0; i < mTargetTypes.size(); ++i) {
            if (mTargetTypes.get(i).isInstance(target)) {
                return true;
            }
        }
    }
    return false;
}
```

``` java
void playTransition(ViewGroup sceneRoot) {
  		//matchStartAndEnd() 筛选出 start scene 和 end scene 中匹配的 transiton value
  		//mStartValuesList 保存 start scene 和 end scene 中同时存在的 transition value 和 只存在 start scene 的 transition value
  		//mEndValuesList 保存 start scene 和 end scene 中同时存在的 transition value 和 只存在 end scene 的 transition value
        mStartValuesList = new ArrayList<TransitionValues>();
        mEndValuesList = new ArrayList<TransitionValues>();
        matchStartAndEnd(mStartValues, mEndValues);

        ArrayMap<Animator, AnimationInfo> runningAnimators = getRunningAnimators();
        int numOldAnims = runningAnimators.size();
        WindowId windowId = sceneRoot.getWindowId();
        for (int i = numOldAnims - 1; i >= 0; i--) {
            Animator anim = runningAnimators.keyAt(i);
            if (anim != null) {
                AnimationInfo oldInfo = runningAnimators.get(anim);
                if (oldInfo != null && oldInfo.view != null && oldInfo.windowId == windowId) {
                    TransitionValues oldValues = oldInfo.values;
                    View oldView = oldInfo.view;
                    TransitionValues startValues = getTransitionValues(oldView, true);
                  	//先取出同时在 mStartValuesList 和 mEndValuesList 中的 结束 transition value 
                    TransitionValues endValues = getMatchedTransitionValues(oldView, true);
                    if (startValues == null && endValues == null) {
                      	//这个 transition value 可能只存在 end scene
                        endValues = mEndValues.viewValues.get(oldView);
                    }
                  	//isTransitionRequired() 通过比较两个 transiton value，返回是否必须创建 transition
                    boolean cancel = (startValues != null || endValues != null) &&
                            oldInfo.transition.isTransitionRequired(oldValues, endValues);
                    if (cancel) {
                      	//如果需要重新创建 transition，取消旧的 anim
                        if (anim.isRunning() || anim.isStarted()) {
                            if (DBG) {
                                Log.d(LOG_TAG, "Canceling anim " + anim);
                            }
                            anim.cancel();
                        } else {
                            if (DBG) {
                                Log.d(LOG_TAG, "removing anim from info list: " + anim);
                            }
                            runningAnimators.remove(anim);
                        }
                    }
                }
            }
        }
		//遍历 mStartValuesList 和 mEndValuesList 来调用 createAnimator() 创建 Animator
        createAnimators(sceneRoot, mStartValues, mEndValues, mStartValuesList, mEndValuesList);
  		//开始运行动画，并清除运行完的动画，并回调 TransitionListener.onTransitionStart() 和 TransitionListener.onTransitionEnd() 
        runAnimators();
    }
```

> `playTransition()` 的逻辑是：
>
> 1. 筛选出起始 transition value 和结束 transition value，两个列表
> 2. 根据是否需要重新创建 transition 的动画，来决定是否取消之前的动画
> 3. 根据 `mStartValuesList` 和 `mEndValuesList` 和 是否需要重新创建 transition 的动画，来调用 `createAnimator()`
> 4. 按顺序执行动画，并清除资源



``` java
//根据起始的 transition value 和结束的 transition value，来决定是否需要创建动画
public boolean isTransitionRequired(@Nullable TransitionValues startValues,
            @Nullable TransitionValues endValues) {
        boolean valuesChanged = false;
        // if startValues null, then transition didn't care to stash values,
        // and won't get canceled
        if (startValues != null && endValues != null) {
          	//如果 getTransitionProperties() 不能为 null
            String[] properties = getTransitionProperties();
            if (properties != null) {
                int count = properties.length;
                for (int i = 0; i < count; i++) {
                    if (isValueChanged(startValues, endValues, properties[i])) {
                        valuesChanged = true;
                        break;
                    }
                }
            } else {
                for (String key : startValues.values.keySet()) {
                    if (isValueChanged(startValues, endValues, key)) {
                        valuesChanged = true;
                        break;
                    }
                }
            }
        }
        return valuesChanged;
    }
```

``` java
 private static boolean isValueChanged(TransitionValues oldValues, TransitionValues newValues,
            String key) {
        if (oldValues.values.containsKey(key) != newValues.values.containsKey(key)) {
            // The transition didn't care about this particular value, so we don't care, either.	
          	//key 只存在其中一个 transition value
            return false;
        }
        Object oldValue = oldValues.values.get(key);
        Object newValue = newValues.values.get(key);
        boolean changed;
        if (oldValue == null && newValue == null) {
            // both are null
            changed = false;
        } else if (oldValue == null || newValue == null) {
            // one is null
          	//其中一个为 null
            changed = true;
        } else {
            // neither is null
          	//比较两个值
            changed = !oldValue.equals(newValue);
        }
        if (DBG && changed) {
            Log.d(LOG_TAG, "Transition.playTransition: " +
                    "oldValue != newValue for " + key +
                    ": old, new = " + oldValue + ", " + newValue);
        }
        return changed;
    }
```

> 这里 `isTransitionRequired()` 方法中的比较，主要是通过对比 transition value 中的 values，同时这个值是在 `captureStartValues()` 和 `captureEndValues()` 中添加的。



``` java
// 根据 transition value 创建动画，并运行动画
protected void createAnimators(ViewGroup sceneRoot, TransitionValuesMaps startValues,
            TransitionValuesMaps endValues, ArrayList<TransitionValues> startValuesList,
            ArrayList<TransitionValues> endValuesList) {
        if (DBG) {
            Log.d(LOG_TAG, "createAnimators() for " + this);
        }
        ArrayMap<Animator, AnimationInfo> runningAnimators = getRunningAnimators();
        long minStartDelay = Long.MAX_VALUE;
        int minAnimator = mAnimators.size();
        SparseLongArray startDelays = new SparseLongArray();
        int startValuesListCount = startValuesList.size();
        for (int i = 0; i < startValuesListCount; ++i) {
            TransitionValues start = startValuesList.get(i);
            TransitionValues end = endValuesList.get(i);
          	//判断下 targetedTransitions 是否包含当前 transition
            if (start != null && !start.targetedTransitions.contains(this)) {
                start = null;
            }
            if (end != null && !end.targetedTransitions.contains(this)) {
                end = null;
            }
            if (start == null && end == null) {
                continue;
            }
            // Only bother trying to animate with values that differ between start/end
            boolean isChanged = start == null || end == null || isTransitionRequired(start, end);
            if (isChanged) {
              	//需要创建动画
                //省略
                // TODO: what to do about targetIds and itemIds
              	//调用 createAnimator() 创建动画
                Animator animator = createAnimator(sceneRoot, start, end);
                if (animator != null) {
                    // Save animation info for future cancellation purposes
                    View view = null;
                    TransitionValues infoValues = null;
                    if (end != null) {
                        view = end.view;
                        String[] properties = getTransitionProperties();
                        if (view != null && properties != null && properties.length > 0) {
                            infoValues = new TransitionValues();
                            infoValues.view = view;
                            TransitionValues newValues = endValues.viewValues.get(view);
                            if (newValues != null) {
                                for (int j = 0; j < properties.length; ++j) {
                                    infoValues.values.put(properties[j],
                                            newValues.values.get(properties[j]));
                                }
                            }
                            int numExistingAnims = runningAnimators.size();
                            for (int j = 0; j < numExistingAnims; ++j) {
                                Animator anim = runningAnimators.keyAt(j);
                                AnimationInfo info = runningAnimators.get(anim);
                                if (info.values != null && info.view == view &&
                                        ((info.name == null && getName() == null) ||
                                                info.name.equals(getName()))) {
                                    if (info.values.equals(infoValues)) {
                                        // Favor the old animator
                                      	//sRunningAnimators 中存在相同效果的动画
                                        animator = null;
                                        break;
                                    }
                                }
                            }
                        }
                    } else {
                        view = (start != null) ? start.view : null;
                    }
                    if (animator != null) {
                        if (mPropagation != null) {
                            long delay = mPropagation
                                    .getStartDelay(sceneRoot, this, start, end);
                            startDelays.put(mAnimators.size(), delay);
                            minStartDelay = Math.min(delay, minStartDelay);
                        }
                        AnimationInfo info = new AnimationInfo(view, getName(), this,
                                sceneRoot.getWindowId(), infoValues);
                      	//添加到需要运行的动画
                        runningAnimators.put(animator, info);
                      	//mAnimators 保存创建的动画
                        mAnimators.add(animator);
                    }
                }
            }
        }
        if (startDelays.size() != 0) {
          	//按顺序开始执行 mAnimators 中的动画
            for (int i = 0; i < startDelays.size(); i++) {
                int index = startDelays.keyAt(i);
                Animator animator = mAnimators.get(index);
                long delay = startDelays.valueAt(i) - minStartDelay + animator.getStartDelay();
                animator.setStartDelay(delay);
            }
        }
    }
```

``` java
protected void runAnimators() {
        if (DBG) {
            Log.d(LOG_TAG, "runAnimators() on " + this);
        }
  		//回调方法
        start();
        ArrayMap<Animator, AnimationInfo> runningAnimators = getRunningAnimators();
        // Now start every Animator that was previously created for this transition
        for (Animator anim : mAnimators) {
            if (DBG) {
                Log.d(LOG_TAG, "  anim: " + anim);
            }
            if (runningAnimators.containsKey(anim)) {
              	//回调方法
                start();
                runAnimator(anim, runningAnimators);
            }
        }
        mAnimators.clear();
  		//回调方法
        end();
    }
```

``` java
private void runAnimator(Animator animator,
            final ArrayMap<Animator, AnimationInfo> runningAnimators) {
        if (animator != null) {
            // TODO: could be a single listener instance for all of them since it uses the param
            animator.addListener(new AnimatorListenerAdapter() {
                @Override
                public void onAnimationStart(Animator animation) {
                    mCurrentAnimators.add(animation);
                }
                @Override
                public void onAnimationEnd(Animator animation) {
                  	//从 sRunningAnimators 中删除
                    runningAnimators.remove(animation);
                    mCurrentAnimators.remove(animation);
                }
            });
          	//执行动画
            animate(animator);
        }
    }
```

``` java
protected void animate(Animator animator) {
        // TODO: maybe pass auto-end as a boolean parameter?
        if (animator == null) {
            end();
        } else {
            if (getDuration() >= 0) {
                animator.setDuration(getDuration());
            }
            if (getStartDelay() >= 0) {
                animator.setStartDelay(getStartDelay() + animator.getStartDelay());
            }
            if (getInterpolator() != null) {
                animator.setInterpolator(getInterpolator());
            }
            animator.addListener(new AnimatorListenerAdapter() {
                @Override
                public void onAnimationEnd(Animator animation) {
                    end();
                    animation.removeListener(this);
                }
            });
            animator.start();
        }
    }
```

``` java
//获取 transition 中的动画信息 
private static ArrayMap<Animator, AnimationInfo> getRunningAnimators() {
        ArrayMap<Animator, AnimationInfo> runningAnimators = sRunningAnimators.get();
        if (runningAnimators == null) {
            runningAnimators = new ArrayMap<Animator, AnimationInfo>();
            sRunningAnimators.set(runningAnimators);
        }
        return runningAnimators;
    }
```



