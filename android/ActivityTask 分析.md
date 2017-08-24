#### 结论

* `launchMode`和`flags`同时存在时，`flags`覆盖`launchMode`。
* 当flags指定了`NEW_TASK`时，`taskAffinity`必须指定与默认值(包名)不同的值，才会新建Task(如果目标Activity实例不存在时)。
* 当flags指定了`NEW_TASK`时，且目标Activity实例存在时，会将目标Task(目标Actvity所在的Task)前置，如果前台Task不为目标Task的话。
* 使用`NEW_TASK`和`SINAGLE_TASK`有少许区别，设置了`SINGLE_TASK`会自动设置`NEW_TASK`，而且如果已经存在目标Activity实例，则会调用`newIntent`，并且会推出前面的Activity。
* `CLEAR_TASK`只能和`NEW_TASK`配合使用，将目标task(根据`taskAffinity`判断)清空，再将目标Activity新实例压入。
* `ClEAR_TOP`的作用是如果当前Task存在目标Activity实例，则该实例前面的Activity推出，然后创建新目标Activity实例(**即使目标Activity已经位于栈顶，也会destroy，重新创建实例**)位于栈顶。
  * 当和`NEW_TASK`配合使用时，则会根据`taskAffinity`寻找目标Task，如果存在目标Activity实例时，则将目标栈中目标Activity实例前面的Activity推出，让创建新目标Activity实例位于栈顶(**即使目标Activity已经位于栈顶**)。否则，则创建目标Activity实例并压入目标Tsk(如果目标Task不存在则创建Task，根据`taskAffinity`寻找目标Task)。
  * 当和`SINGLE_TOP`配合使用时，如果当前Task存在目标Activity实例时，则该实例前面的Activity推出，让该实例位于栈顶(**注意区别和`CLEAR_TOP`单独使用，关键在于有没有创建新实例**)。
  * 当和`NEW_TASK`，`SINGLE_TOP`一起配合使用时，如果目标Activity实例存在时，则将目标Task中位于目标Activity前面的Activity推出，让该实例(不重新创建实例)位于栈顶。


#### 源码分析

* `TaskRecord`：记录Task的信息
* `ActivityRecord`：记录Activity信息
* `ActivityStack`：存放Activity的栈
* `ActivityStackSupervisor`：对ActivityStack管理
* `ProcessRecord`：记录进程的信息
* `ResolveInfo`：从IntentFilter解析Intent返回的信息
* `ActivityInfo`：可以获取特定Activity或Receiver的信息

``` java
//com.android.server.am.ActivityStarter
final int startActivityLocked(...) {
        int err = ActivityManager.START_SUCCESS;

        ProcessRecord callerApp = null;
        if (caller != null) {
          	//获取进程信息
            callerApp = mService.getRecordForAppLocked(caller);
            if (callerApp != null) {
              	//记录进程pid,uid
                callingPid = callerApp.pid;
                callingUid = callerApp.info.uid;
            } else {
                Slog.w(TAG, "Unable to find app for caller " + caller
                        + " (pid=" + callingPid + ") when starting: "
                        + intent.toString());
                err = ActivityManager.START_PERMISSION_DENIED;
            }
        }

        final int userId = aInfo != null ? UserHandle.getUserId(aInfo.applicationInfo.uid) : 0;

        if (err == ActivityManager.START_SUCCESS) {
            Slog.i(TAG, "START u" + userId + " {" + intent.toShortString(true, true, true, false)
                    + "} from uid " + callingUid
                    + " on display " + (container == null ? (mSupervisor.mFocusedStack == null ?
                    Display.DEFAULT_DISPLAY : mSupervisor.mFocusedStack.mDisplayId) :
                    (container.mActivityDisplay == null ? Display.DEFAULT_DISPLAY :
                            container.mActivityDisplay.mDisplayId)));
        }

        ActivityRecord sourceRecord = null;
        ActivityRecord resultRecord = null;
        if (resultTo != null) {
          	//根据resultTo获取源Activity记录
            sourceRecord = mSupervisor.isInAnyStackLocked(resultTo);
            if (DEBUG_RESULTS) Slog.v(TAG_RESULTS,
                    "Will send result to " + resultTo + " " + sourceRecord);
            if (sourceRecord != null) {
              	//是否有requestCode
                if (requestCode >= 0 && !sourceRecord.finishing) {
                    resultRecord = sourceRecord;
                }
            }
        }

        final int launchFlags = intent.getFlags();

        if ((launchFlags & Intent.FLAG_ACTIVITY_FORWARD_RESULT) != 0 && sourceRecord != null) {
          	//FORWARD_RESULT的作用：将对源Activity的requestCode，传递给目标Activity，让目标Activity结束时能将setResul返回给设置requestCode的起始Activity
            // Transfer the result target from the source activity to the new
            // one being started, including any failures.
            if (requestCode >= 0) {
                ActivityOptions.abort(options);
                return ActivityManager.START_FORWARD_AND_REQUEST_CONFLICT;
            }
            resultRecord = sourceRecord.resultTo;
            if (resultRecord != null && !resultRecord.isInStackLocked()) {
                resultRecord = null;
            }
            resultWho = sourceRecord.resultWho;
            requestCode = sourceRecord.requestCode;
            sourceRecord.resultTo = null;
            if (resultRecord != null) {
                resultRecord.removeResultsLocked(sourceRecord, resultWho, requestCode);
            }
            if (sourceRecord.launchedFromUid == callingUid) {
                // The new activity is being launched from the same uid as the previous
                // activity in the flow, and asking to forward its result back to the
                // previous.  In this case the activity is serving as a trampoline between
                // the two, so we also want to update its launchedFromPackage to be the
                // same as the previous activity.  Note that this is safe, since we know
                // these two packages come from the same uid; the caller could just as
                // well have supplied that same package name itself.  This specifially
                // deals with the case of an intent picker/chooser being launched in the app
                // flow to redirect to an activity picked by the user, where we want the final
                // activity to consider it to have been launched by the previous app activity.
                callingPackage = sourceRecord.launchedFromPackage;
            }
        }

        if (err == ActivityManager.START_SUCCESS && intent.getComponent() == null) {
            // We couldn't find a class that can handle the given Intent.
            // That's the end of that!
          	//当前Intent没有得到响应
            err = ActivityManager.START_INTENT_NOT_RESOLVED;
        }

        if (err == ActivityManager.START_SUCCESS && aInfo == null) {
            // We couldn't find the specific class specified in the Intent.
            // Also the end of the line.
          	//当前Intent显式指定的class不存在。
            err = ActivityManager.START_CLASS_NOT_FOUND;
        }

        if (err == ActivityManager.START_SUCCESS && sourceRecord != null
                && sourceRecord.task.voiceSession != null) {
          	//语音会话处理
            ...
        }

        if (err == ActivityManager.START_SUCCESS && voiceSession != null) {
            //语音会话处理
        }

        final ActivityStack resultStack = resultRecord == null ? null : resultRecord.task.stack;

        if (err != START_SUCCESS) {
          	//启动失败
            if (resultRecord != null) {
                resultStack.sendActivityResultLocked(
                        -1, resultRecord, resultWho, requestCode, RESULT_CANCELED, null);
            }
            ActivityOptions.abort(options);
            return err;
        }
		
  		//权限检查
  		//是否中止
        boolean abort = !mSupervisor.checkStartAnyActivityPermission(intent, aInfo, resultWho,
                requestCode, callingPid, callingUid, callingPackage, ignoreTargetSecurity, callerApp,
                resultRecord, resultStack, options);
        abort |= !mService.mIntentFirewall.checkStartActivity(intent, callingUid,
                callingPid, resolvedType, aInfo.applicationInfo);

        if (mService.mController != null) {
            try {
                // The Intent we give to the watcher has the extra data
                // stripped off, since it can contain private information.
              	//是否中止
                Intent watchIntent = intent.cloneFilter();
                abort |= !mService.mController.activityStarting(watchIntent,
                        aInfo.applicationInfo.packageName);
            } catch (RemoteException e) {
                mService.mController = null;
            }
        }
		
        mInterceptor.setStates(userId, realCallingPid, realCallingUid, startFlags, callingPackage);
  		//回调ActivityStartInterceptor.intercept
  		//Activity启动拦截器
        mInterceptor.intercept(intent, rInfo, aInfo, resolvedType, inTask, callingPid, callingUid,
                options);
  		//接收拦截器的处理结果
        intent = mInterceptor.mIntent;
        rInfo = mInterceptor.mRInfo;
        aInfo = mInterceptor.mAInfo;
        resolvedType = mInterceptor.mResolvedType;
        inTask = mInterceptor.mInTask;
        callingPid = mInterceptor.mCallingPid;
        callingUid = mInterceptor.mCallingUid;
        options = mInterceptor.mActivityOptions;
        if (abort) {
          	//如果中止
            if (resultRecord != null) {
                resultStack.sendActivityResultLocked(-1, resultRecord, resultWho, requestCode,
                        RESULT_CANCELED, null);
            }
            // We pretend to the caller that it was really started, but
            // they will just get a cancel result.
            ActivityOptions.abort(options);
            return START_SUCCESS;
        }

        // If permissions need a review before any of the app components can run, we
        // launch the review activity and pass a pending intent to start the activity
        // we are to launching now after the review is completed.
        if (Build.PERMISSIONS_REVIEW_REQUIRED && aInfo != null) {
          	//检查运行时权限
            if (mService.getPackageManagerInternalLocked().isPermissionsReviewRequired(
                    aInfo.packageName, userId)) {
                IIntentSender target = mService.getIntentSenderLocked(
                        ActivityManager.INTENT_SENDER_ACTIVITY, callingPackage,
                        callingUid, userId, null, null, 0, new Intent[]{intent},
                        new String[]{resolvedType}, PendingIntent.FLAG_CANCEL_CURRENT
                                | PendingIntent.FLAG_ONE_SHOT, null);

                final int flags = intent.getFlags();
                Intent newIntent = new Intent(Intent.ACTION_REVIEW_PERMISSIONS);
                newIntent.setFlags(flags
                        | Intent.FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS);
                newIntent.putExtra(Intent.EXTRA_PACKAGE_NAME, aInfo.packageName);
                newIntent.putExtra(Intent.EXTRA_INTENT, new IntentSender(target));
                if (resultRecord != null) {
                    newIntent.putExtra(Intent.EXTRA_RESULT_NEEDED, true);
                }
                intent = newIntent;

                resolvedType = null;
                callingUid = realCallingUid;
                callingPid = realCallingPid;

                rInfo = mSupervisor.resolveIntent(intent, resolvedType, userId);
                aInfo = mSupervisor.resolveActivity(intent, rInfo, startFlags,
                        null /*profilerInfo*/);

                if (DEBUG_PERMISSIONS_REVIEW) {
                    Slog.i(TAG, "START u" + userId + " {" + intent.toShortString(true, true,
                            true, false) + "} from uid " + callingUid + " on display "
                            + (container == null ? (mSupervisor.mFocusedStack == null ?
                            Display.DEFAULT_DISPLAY : mSupervisor.mFocusedStack.mDisplayId) :
                            (container.mActivityDisplay == null ? Display.DEFAULT_DISPLAY :
                                    container.mActivityDisplay.mDisplayId)));
                }
            }
        }

        // If we have an ephemeral app, abort the process of launching the resolved intent.
        // Instead, launch the ephemeral installer. Once the installer is finished, it
        // starts either the intent we resolved here [on install error] or the ephemeral
        // app [on install success].
        if (rInfo != null && rInfo.ephemeralResolveInfo != null) {
          	//短暂的应用程序
          	//启动安装
            intent = buildEphemeralInstallerIntent(intent, ephemeralIntent,
                    rInfo.ephemeralResolveInfo.getPackageName(), callingPackage, resolvedType,
                    userId);
            resolvedType = null;
            callingUid = realCallingUid;
            callingPid = realCallingPid;

            aInfo = mSupervisor.resolveActivity(intent, rInfo, startFlags, null /*profilerInfo*/);
        }
		
  		//创建目标ActivityRecord
        ActivityRecord r = new ActivityRecord(mService, callerApp, callingUid, callingPackage,
                intent, resolvedType, aInfo, mService.mConfiguration, resultRecord, resultWho,
                requestCode, componentSpecified, voiceSession != null, mSupervisor, container,
                options, sourceRecord);
  		//保存在输出Activiy中
        if (outActivity != null) {
            outActivity[0] = r;
        }

        if (r.appTimeTracker == null && sourceRecord != null) {
            // If the caller didn't specify an explicit time tracker, we want to continue
            // tracking under any it has.
          	//默认TimeTracker
            r.appTimeTracker = sourceRecord.appTimeTracker;
        }

        final ActivityStack stack = mSupervisor.mFocusedStack;
  		//mResumedActivity：该ActivityStack中处于活动的Activity
        if (voiceSession == null && (stack.mResumedActivity == null
                || stack.mResumedActivity.info.applicationInfo.uid != callingUid)) {
            if (!mService.checkAppSwitchAllowedLocked(callingPid, callingUid,
                    realCallingPid, realCallingUid, "Activity start")) {
              	//启动一个PendingActivity
                PendingActivityLaunch pal =  new PendingActivityLaunch(r,
                        sourceRecord, startFlags, stack, callerApp);
                mPendingActivityLaunches.add(pal);
                ActivityOptions.abort(options);
                return ActivityManager.START_SWITCHES_CANCELED;
            }
        }

        if (mService.mDidAppSwitch) {
            // This is the second allowed switch since we stopped switches,
            // so now just generally allow switches.  Use case: user presses
            // home (switches disabled, switch to home, mDidAppSwitch now true);
            // user taps a home icon (coming from home so allowed, we hit here
            // and now allow anyone to switch again).
            mService.mAppSwitchesAllowedTime = 0;
        } else {
            mService.mDidAppSwitch = true;
        }

        doPendingActivityLaunchesLocked(false);

        try {
            mService.mWindowManager.deferSurfaceLayout();
            err = startActivityUnchecked(r, sourceRecord, voiceSession, voiceInteractor, startFlags,
                    true, options, inTask);
        } finally {
            mService.mWindowManager.continueSurfaceLayout();
        }
        postStartActivityUncheckedProcessing(r, err, stack.mStackId, mSourceRecord, mTargetStack);
        return err;
    }
```

``` java
private int startActivityUnchecked(...) {
		//设置一些初始化状态
  		//比如从AndroidManifests中读取launchMode的值等等
        setInitialState(...);
		
  		//根据launchMode设置flags
        computeLaunchingTaskFlags();
		
  		//设置源Stack
        computeSourceStack();

        mIntent.setFlags(mLaunchFlags);
		//获取是否可以复用的Activity
  		//注意：这里是返回匹配的Task的栈顶Activity
  		//如果目标Activity设置了NEW_TASK且目标Task和目标Activity已存在，而且不是位于栈顶，则返回栈顶元素
        mReusedActivity = getReusableIntentActivity();

        final int preferredLaunchStackId =
                (mOptions != null) ? mOptions.getLaunchStackId() : INVALID_STACK_ID;

        if (mReusedActivity != null) {
          	//如果存在可以复用的Activity
            // When the flags NEW_TASK and CLEAR_TASK are set, then the task gets reused but
            // still needs to be a lock task mode violation since the task gets cleared out and
            // the device would otherwise leave the locked task.
            if (mSupervisor.isLockTaskModeViolation(mReusedActivity.task,
                    (mLaunchFlags & (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK))
                            == (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK))) {
                mSupervisor.showLockTaskToast();
                Slog.e(TAG, "startActivityUnchecked: Attempt to violate Lock Task Mode");
                return START_RETURN_LOCK_TASK_MODE_VIOLATION;
            }

            if (mStartActivity.task == null) {
                mStartActivity.task = mReusedActivity.task;
            }
            if (mReusedActivity.task.intent == null) {
                // This task was started because of movement of the activity based on affinity...
                // Now that we are actually launching it, we can assign the base intent.
                mReusedActivity.task.setIntent(mStartActivity);
            }

            // This code path leads to delivering a new intent, we want to make sure we schedule it
            // as the first operation, in case the activity will be resumed as a result of later
            // operations.
            if ((mLaunchFlags & FLAG_ACTIVITY_CLEAR_TOP) != 0
                    || mLaunchSingleInstance || mLaunchSingleTask) {
                // In this situation we want to remove all activities from the task up to the one
                // being started. In most cases this means we are resetting the task to its initial
                // state.
                final ActivityRecord top = mReusedActivity.task.performClearTaskForReuseLocked(
                        mStartActivity, mLaunchFlags);
                if (top != null) {
                    if (top.frontOfTask) {
                        // Activity aliases may mean we use different intents for the top activity,
                        // so make sure the task now has the identity of the new intent.
                        top.task.setIntent(mStartActivity);
                    }
                    ActivityStack.logStartActivity(AM_NEW_INTENT, mStartActivity, top.task);
                    top.deliverNewIntentLocked(mCallingUid, mStartActivity.intent,
                            mStartActivity.launchedFromPackage);
                }
            }

            sendPowerHintForLaunchStartIfNeeded(false /* forceSend */);

            mReusedActivity = setTargetStackAndMoveToFrontIfNeeded(mReusedActivity);

            if ((mStartFlags & START_FLAG_ONLY_IF_NEEDED) != 0) {
                // We don't need to start a new activity, and the client said not to do anything
                // if that is the case, so this is it!  And for paranoia, make sure we have
                // correctly resumed the top activity.
                resumeTargetStackIfNeeded();
                return START_RETURN_INTENT_TO_CALLER;
            }
            setTaskFromIntentActivity(mReusedActivity);

            if (!mAddingToTask && mReuseTask == null) {
                // We didn't do anything...  but it was needed (a.k.a., client don't use that
                // intent!)  And for paranoia, make sure we have correctly resumed the top activity.
                resumeTargetStackIfNeeded();
                return START_TASK_TO_FRONT;
            }
        }

        if (mStartActivity.packageName == null) {
            if (mStartActivity.resultTo != null && mStartActivity.resultTo.task.stack != null) {
                mStartActivity.resultTo.task.stack.sendActivityResultLocked(
                        -1, mStartActivity.resultTo, mStartActivity.resultWho,
                        mStartActivity.requestCode, RESULT_CANCELED, null);
            }
            ActivityOptions.abort(mOptions);
            return START_CLASS_NOT_FOUND;
        }

        // If the activity being launched is the same as the one currently at the top, then
        // we need to check if it should only be launched once.
        final ActivityStack topStack = mSupervisor.mFocusedStack;
        final ActivityRecord top = topStack.topRunningNonDelayedActivityLocked(mNotTop);
  		//判断是否可以不重新启动
  		//判断关键：
  		//1.栈顶Activity等于目标Activity，而且设置了(SINGLE_TOP或SINGLETOP或SingleTask)
        final boolean dontStart = top != null && mStartActivity.resultTo == null
                && top.realActivity.equals(mStartActivity.realActivity)
                && top.userId == mStartActivity.userId
                && top.app != null && top.app.thread != null
                && ((mLaunchFlags & FLAG_ACTIVITY_SINGLE_TOP) != 0
                || mLaunchSingleTop || mLaunchSingleTask);
        if (dontStart) {
            ActivityStack.logStartActivity(AM_NEW_INTENT, top, top.task);
            // For paranoia, make sure we have correctly resumed the top activity.
            topStack.mLastPausedActivity = null;
            if (mDoResume) {
                mSupervisor.resumeFocusedStackTopActivityLocked();
            }
            ActivityOptions.abort(mOptions);
            if ((mStartFlags & START_FLAG_ONLY_IF_NEEDED) != 0) {
                // We don't need to start a new activity, and the client said not to do
                // anything if that is the case, so this is it!
                return START_RETURN_INTENT_TO_CALLER;
            }
            top.deliverNewIntentLocked(
                    mCallingUid, mStartActivity.intent, mStartActivity.launchedFromPackage);

            // Don't use mStartActivity.task to show the toast. We're not starting a new activity
            // but reusing 'top'. Fields in mStartActivity may not be fully initialized.
            mSupervisor.handleNonResizableTaskIfNeeded(
                    top.task, preferredLaunchStackId, topStack.mStackId);

            return START_DELIVERED_TO_TOP;
        }

        boolean newTask = false;
        final TaskRecord taskToAffiliate = (mLaunchTaskBehind && mSourceRecord != null)
                ? mSourceRecord.task : null;

        // Should this be considered a new task?
        if (mStartActivity.resultTo == null && mInTask == null && !mAddingToTask
                && (mLaunchFlags & FLAG_ACTIVITY_NEW_TASK) != 0) {
            newTask = true;
          	//创建或复用task
            setTaskFromReuseOrCreateNewTask(taskToAffiliate);

            if (mSupervisor.isLockTaskModeViolation(mStartActivity.task)) {
                Slog.e(TAG, "Attempted Lock Task Mode violation mStartActivity=" + mStartActivity);
                return START_RETURN_LOCK_TASK_MODE_VIOLATION;
            }
            if (!mMovedOtherTask) {
                // If stack id is specified in activity options, usually it means that activity is
                // launched not from currently focused stack (e.g. from SysUI or from shell) - in
                // that case we check the target stack.
                updateTaskReturnToType(mStartActivity.task, mLaunchFlags,
                        preferredLaunchStackId != INVALID_STACK_ID ? mTargetStack : topStack);
            }
        } else if (mSourceRecord != null) {
            if (mSupervisor.isLockTaskModeViolation(mSourceRecord.task)) {
                Slog.e(TAG, "Attempted Lock Task Mode violation mStartActivity=" + mStartActivity);
                return START_RETURN_LOCK_TASK_MODE_VIOLATION;
            }

            final int result = setTaskFromSourceRecord();
            if (result != START_SUCCESS) {
                return result;
            }
        } else if (mInTask != null) {
            // The caller is asking that the new activity be started in an explicit
            // task it has provided to us.
            if (mSupervisor.isLockTaskModeViolation(mInTask)) {
                Slog.e(TAG, "Attempted Lock Task Mode violation mStartActivity=" + mStartActivity);
                return START_RETURN_LOCK_TASK_MODE_VIOLATION;
            }

            final int result = setTaskFromInTask();
            if (result != START_SUCCESS) {
                return result;
            }
        } else {
            // This not being started from an existing activity, and not part of a new task...
            // just put it in the top task, though these days this case should never happen.
            setTaskToCurrentTopOrCreateNewTask();
        }

        mService.grantUriPermissionFromIntentLocked(mCallingUid, mStartActivity.packageName,
                mIntent, mStartActivity.getUriPermissionsLocked(), mStartActivity.userId);

        if (mSourceRecord != null && mSourceRecord.isRecentsActivity()) {
            mStartActivity.task.setTaskToReturnTo(RECENTS_ACTIVITY_TYPE);
        }
        if (newTask) {
            EventLog.writeEvent(
                    EventLogTags.AM_CREATE_TASK, mStartActivity.userId, mStartActivity.task.taskId);
        }
        ActivityStack.logStartActivity(
                EventLogTags.AM_CREATE_ACTIVITY, mStartActivity, mStartActivity.task);
        mTargetStack.mLastPausedActivity = null;

        sendPowerHintForLaunchStartIfNeeded(false /* forceSend */);
		//在目标Task中去启动目标Activity
        mTargetStack.startActivityLocked(mStartActivity, newTask, mKeepCurTransition, mOptions);
        if (mDoResume) {
            if (!mLaunchTaskBehind) {
                // TODO(b/26381750): Remove this code after verification that all the decision
                // points above moved targetStack to the front which will also set the focus
                // activity.
                mService.setFocusedActivityLocked(mStartActivity, "startedActivity");
            }
            final ActivityRecord topTaskActivity = mStartActivity.task.topRunningActivityLocked();
            if (!mTargetStack.isFocusable()
                    || (topTaskActivity != null && topTaskActivity.mTaskOverlay
                    && mStartActivity != topTaskActivity)) {
                // If the activity is not focusable, we can't resume it, but still would like to
                // make sure it becomes visible as it starts (this will also trigger entry
                // animation). An example of this are PIP activities.
                // Also, we don't want to resume activities in a task that currently has an overlay
                // as the starting activity just needs to be in the visible paused state until the
                // over is removed.
                mTargetStack.ensureActivitiesVisibleLocked(null, 0, !PRESERVE_WINDOWS);
                // Go ahead and tell window manager to execute app transition for this activity
                // since the app transition will not be triggered through the resume channel.
                mWindowManager.executeAppTransition();
            } else {
                mSupervisor.resumeFocusedStackTopActivityLocked(mTargetStack, mStartActivity,
                        mOptions);
            }
        } else {
            mTargetStack.addRecentActivityLocked(mStartActivity);
        }
        mSupervisor.updateUserStackLocked(mStartActivity.userId, mTargetStack);

        mSupervisor.handleNonResizableTaskIfNeeded(
                mStartActivity.task, preferredLaunchStackId, mTargetStack.mStackId);

        return START_SUCCESS;
    }
```

``` java
private void setInitialState(...) {
  		//重置值
        reset();
		//保存值
        mStartActivity = r;
        mIntent = r.intent;
        mOptions = options;
        mCallingUid = r.launchedFromUid;
        mSourceRecord = sourceRecord;
        mVoiceSession = voiceSession;
        mVoiceInteractor = voiceInteractor;

        mLaunchBounds = getOverrideBounds(r, options, inTask);
		//保存在AndroidManifests中配置的Activity launchMode
        mLaunchSingleTop = r.launchMode == LAUNCH_SINGLE_TOP;
        mLaunchSingleInstance = r.launchMode == LAUNCH_SINGLE_INSTANCE;
        mLaunchSingleTask = r.launchMode == LAUNCH_SINGLE_TASK;
  		//所有涉及到Document标记的都是对概览屏幕(recentes)的处理
        mLaunchFlags = adjustLaunchFlagsToDocumentMode(
                r, mLaunchSingleInstance, mLaunchSingleTask, mIntent.getFlags());
        mLaunchTaskBehind = r.mLaunchTaskBehind
                && !mLaunchSingleTask && !mLaunchSingleInstance
                && (mLaunchFlags & FLAG_ACTIVITY_NEW_DOCUMENT) != 0;
		
  		//如果目标Activity需要返回结果(setResult)，而目标Activity又在新的Task中启动，则发送取消的结果(setResult(RESULT_CANCELED))
        sendNewTaskResultRequestIfNeeded();

        if ((mLaunchFlags & FLAG_ACTIVITY_NEW_DOCUMENT) != 0 && r.resultTo == null) {
          	//如果设置了NEW_DOCUMENT又不需要返回结果，添加NEW_TASK
            mLaunchFlags |= FLAG_ACTIVITY_NEW_TASK;
        }

        // If we are actually going to launch in to a new task, there are some cases where
        // we further want to do multiple task.
        if ((mLaunchFlags & FLAG_ACTIVITY_NEW_TASK) != 0) {
            if (mLaunchTaskBehind
                    || r.info.documentLaunchMode == DOCUMENT_LAUNCH_ALWAYS) {
                mLaunchFlags |= FLAG_ACTIVITY_MULTIPLE_TASK;
            }
        }

        // We'll invoke onUserLeaving before onPause only if the launching
        // activity did not explicitly state that this is an automated launch.
        mSupervisor.mUserLeaving = (mLaunchFlags & FLAG_ACTIVITY_NO_USER_ACTION) == 0;
        if (DEBUG_USER_LEAVING) Slog.v(TAG_USER_LEAVING,
                "startActivity() => mUserLeaving=" + mSupervisor.mUserLeaving);

        // If the caller has asked not to resume at this point, we make note
        // of this in the record so that we can skip it when trying to find
        // the top running activity.
  		//是否设置为活动状态
        mDoResume = doResume;
        if (!doResume || !mSupervisor.okToShowLocked(r)) {
            r.delayedResume = true;
            mDoResume = false;
        }

        if (mOptions != null && mOptions.getLaunchTaskId() != -1 && mOptions.getTaskOverlay()) {
          	//存在ActivityOptions
            r.mTaskOverlay = true;
            final TaskRecord task = mSupervisor.anyTaskForIdLocked(mOptions.getLaunchTaskId());
            final ActivityRecord top = task != null ? task.getTopActivity() : null;
            if (top != null && !top.visible) {

                // The caller specifies that we'd like to be avoided to be moved to the front, so be
                // it!
                mDoResume = false;
                mAvoidMoveToFront = true;
            }
        }
		//不压入到栈顶的Activity
        mNotTop = (mLaunchFlags & FLAG_ACTIVITY_PREVIOUS_IS_TOP) != 0 ? r : null;

        mInTask = inTask;
        // In some flows in to this function, we retrieve the task record and hold on to it
        // without a lock before calling back in to here...  so the task at this point may
        // not actually be in recents.  Check for that, and if it isn't in recents just
        // consider it invalid.
        if (inTask != null && !inTask.inRecents) {
            Slog.w(TAG, "Starting activity in task not in recents: " + inTask);
            mInTask = null;
        }

        mStartFlags = startFlags;
        // If the onlyIfNeeded flag is set, then we can do this if the activity being launched
        // is the same as the one making the call...  or, as a special case, if we do not know
        // the caller then we count the current top activity as the caller.
        if ((startFlags & START_FLAG_ONLY_IF_NEEDED) != 0) {
          	//处理START_FLAG_ONLY_IF_NEEDED
            ActivityRecord checkedCaller = sourceRecord;
            if (checkedCaller == null) {
                checkedCaller = mSupervisor.mFocusedStack.topRunningNonDelayedActivityLocked(
                        mNotTop);
            }
            if (!checkedCaller.realActivity.equals(r.realActivity)) {
                // Caller is not the same as launcher, so always needed.
                mStartFlags &= ~START_FLAG_ONLY_IF_NEEDED;
            }
        }
		//是否不需要动画
        mNoAnimation = (mLaunchFlags & FLAG_ACTIVITY_NO_ANIMATION) != 0;
    }
```

``` java
private void computeLaunchingTaskFlags() {
        // If the caller is not coming from another activity, but has given us an explicit task into
        // which they would like us to launch the new activity, then let's see about doing that.
        if (mSourceRecord == null && mInTask != null && mInTask.stack != null) {
          	//如果调用不是一个Activity
            final Intent baseIntent = mInTask.getBaseIntent();
            final ActivityRecord root = mInTask.getRootActivity();
            if (baseIntent == null) {
                ActivityOptions.abort(mOptions);
                throw new IllegalArgumentException("Launching into task without base intent: "
                        + mInTask);
            }

            // If this task is empty, then we are adding the first activity -- it
            // determines the root, and must be launching as a NEW_TASK.
            if (mLaunchSingleInstance || mLaunchSingleTask) {
                if (!baseIntent.getComponent().equals(mStartActivity.intent.getComponent())) {
                    ActivityOptions.abort(mOptions);
                    throw new IllegalArgumentException("Trying to launch singleInstance/Task "
                            + mStartActivity + " into different task " + mInTask);
                }
                if (root != null) {
                    ActivityOptions.abort(mOptions);
                    throw new IllegalArgumentException("Caller with mInTask " + mInTask
                            + " has root " + root + " but target is singleInstance/Task");
                }
            }

            // If task is empty, then adopt the interesting intent launch flags in to the
            // activity being started.
            if (root == null) {
                final int flagsOfInterest = FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_MULTIPLE_TASK
                        | FLAG_ACTIVITY_NEW_DOCUMENT | FLAG_ACTIVITY_RETAIN_IN_RECENTS;
                mLaunchFlags = (mLaunchFlags & ~flagsOfInterest)
                        | (baseIntent.getFlags() & flagsOfInterest);
                mIntent.setFlags(mLaunchFlags);
                mInTask.setIntent(mStartActivity);
                mAddingToTask = true;

                // If the task is not empty and the caller is asking to start it as the root of
                // a new task, then we don't actually want to start this on the task. We will
                // bring the task to the front, and possibly give it a new intent.
            } else if ((mLaunchFlags & FLAG_ACTIVITY_NEW_TASK) != 0) {
                mAddingToTask = false;

            } else {
                mAddingToTask = true;
            }

            mReuseTask = mInTask;
        } else {
            mInTask = null;
            // Launch ResolverActivity in the source task, so that it stays in the task bounds
            // when in freeform workspace.
            // Also put noDisplay activities in the source task. These by itself can be placed
            // in any task/stack, however it could launch other activities like ResolverActivity,
            // and we want those to stay in the original task.
          	//如果目标Activity是ResolverActivity
          	//当隐式调用有多个响应时，会出现ResolverActivity让用户选择
            if ((mStartActivity.isResolverActivity() || mStartActivity.noDisplay) && mSourceRecord != null
                    && mSourceRecord.isFreeform())  {
                mAddingToTask = true;
            }
        }

        if (mInTask == null) {
            if (mSourceRecord == null) {
              	//目标Activity没有起始Activity，开始新的Task
                // This activity is not being started from another...  in this
                // case we -always- start a new task.
                if ((mLaunchFlags & FLAG_ACTIVITY_NEW_TASK) == 0 && mInTask == null) {
                    Slog.w(TAG, "startActivity called from non-Activity context; forcing " +
                            "Intent.FLAG_ACTIVITY_NEW_TASK for: " + mIntent);
                    mLaunchFlags |= FLAG_ACTIVITY_NEW_TASK;
                }
            } else if (mSourceRecord.launchMode == LAUNCH_SINGLE_INSTANCE) {
              	//如果起始Activity是SINGLE_INSTANCE
                // The original activity who is starting us is running as a single
                // instance...  this new activity it is starting must go on its
                // own task.
                mLaunchFlags |= FLAG_ACTIVITY_NEW_TASK;
            } else if (mLaunchSingleInstance || mLaunchSingleTask) {
                // The activity being started is a single instance...  it always
                // gets launched into its own task.
              	//launchMode配置了SingleInstance或SingleTask，flag增加NewTask
                mLaunchFlags |= FLAG_ACTIVITY_NEW_TASK;
            }
        }
    }
```

``` java
private ActivityRecord getReusableIntentActivity() {
        // We may want to try to place the new activity in to an existing task.  We always
        // do this if the target activity is singleTask or singleInstance; we will also do
        // this if NEW_TASK has been requested, and there is not an additional qualifier telling
        // us to still place it in a new task: multi task, always doc mode, or being asked to
        // launch this as a new task behind the current one
  		//是否可以压入到已存在的栈
        boolean putIntoExistingTask = ((mLaunchFlags & FLAG_ACTIVITY_NEW_TASK) != 0 &&
                (mLaunchFlags & FLAG_ACTIVITY_MULTIPLE_TASK) == 0)
                || mLaunchSingleInstance || mLaunchSingleTask;
        // If bring to front is requested, and no result is requested and we have not been given
        // an explicit task to launch in to, and we can find a task that was started with this
        // same component, then instead of launching bring that one to the front.
        putIntoExistingTask &= mInTask == null && mStartActivity.resultTo == null;
        ActivityRecord intentActivity = null;
        if (mOptions != null && mOptions.getLaunchTaskId() != -1) {
            final TaskRecord task = mSupervisor.anyTaskForIdLocked(mOptions.getLaunchTaskId());
            intentActivity = task != null ? task.getTopActivity() : null;
        } else if (putIntoExistingTask) {
            if (mLaunchSingleInstance) {
                // There can be one and only one instance of single instance activity in the
                // history, and it is always in its own unique task, so we do a special search.			
              	//如果设置了SingleInstance
               intentActivity = mSupervisor.findActivityLocked(mIntent, mStartActivity.info, false);
            } else if ((mLaunchFlags & FLAG_ACTIVITY_LAUNCH_ADJACENT) != 0) {
              	//多屏
                // For the launch adjacent case we only want to put the activity in an existing
                // task if the activity already exists in the history.
                intentActivity = mSupervisor.findActivityLocked(mIntent, mStartActivity.info,
                        !mLaunchSingleTask);
            } else {
                // Otherwise find the best task to put the activity in.
              	//根据目标Activity寻找Task
                intentActivity = mSupervisor.findTaskLocked(mStartActivity);
            }
        }
        return intentActivity;
    }
```

``` java
//com.android.server.am.ActivityStackSupervisor.java
ActivityRecord findTaskLocked(ActivityRecord r) {
        mTmpFindTaskResult.r = null;
        mTmpFindTaskResult.matchedByRootAffinity = false;
        if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Looking for task of " + r);
        for (int displayNdx = mActivityDisplays.size() - 1; displayNdx >= 0; --displayNdx) {
            final ArrayList<ActivityStack> stacks = mActivityDisplays.valueAt(displayNdx).mStacks;
            for (int stackNdx = stacks.size() - 1; stackNdx >= 0; --stackNdx) {
                final ActivityStack stack = stacks.get(stackNdx);
                if (!r.isApplicationActivity() && !stack.isHomeStack()) {
                    if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Skipping stack: (home activity) " + stack);
                    continue;
                }
                if (!stack.mActivityContainer.isEligibleForNewTasks()) {
                    if (DEBUG_TASKS) Slog.d(TAG_TASKS,
                            "Skipping stack: (new task not allowed) " + stack);
                    continue;
                }
              	//遍历已存在的栈，寻找和目标Activity匹配的栈的栈顶Activity
                stack.findTaskLocked(r, mTmpFindTaskResult);
                // It is possible to have task in multiple stacks with the same root affinity.
                // If the match we found was based on root affinity we keep on looking to see if
                // there is a better match in another stack. We eventually return the match based
                // on root affinity if we don't find a better match.
                if (mTmpFindTaskResult.r != null && !mTmpFindTaskResult.matchedByRootAffinity) {
                  	//返回匹配的栈顶Activity
                    return mTmpFindTaskResult.r;
                }
            }
        }
        if (DEBUG_TASKS && mTmpFindTaskResult.r == null) Slog.d(TAG_TASKS, "No task found");
        return mTmpFindTaskResult.r;
    }
```

``` java
//com.android.server.am.ActivityStack.java
//寻找和目标Activity匹配的Task的栈顶Activity
void findTaskLocked(ActivityRecord target, FindTaskResult result) {
        Intent intent = target.intent;
        ActivityInfo info = target.info;
        ComponentName cls = intent.getComponent();
        if (info.targetActivity != null) {
            cls = new ComponentName(info.packageName, info.targetActivity);
        }
        final int userId = UserHandle.getUserId(info.applicationInfo.uid);
        boolean isDocument = intent != null & intent.isDocument();
        // If documentData is non-null then it must match the existing task data.
        Uri documentData = isDocument ? intent.getData() : null;

        if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Looking for task of " + target + " in " + this);
        for (int taskNdx = mTaskHistory.size() - 1; taskNdx >= 0; --taskNdx) {
          	//遍历已存在的Task
            final TaskRecord task = mTaskHistory.get(taskNdx);
            if (task.voiceSession != null) {
                // We never match voice sessions; those always run independently.
                if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Skipping " + task + ": voice session");
                continue;
            }
            if (task.userId != userId) {
                // Looking for a different task.
                if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Skipping " + task + ": different user");
                continue;
            }
          	//获取栈顶Activity
            final ActivityRecord r = task.getTopActivity();
            if (r == null || r.finishing || r.userId != userId ||
                    r.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE) {
              	//如果该栈顶Activity是SINGLE_INSTANCE，则跳过
                if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Skipping " + task + ": mismatch root " + r);
                continue;
            }
            if (r.mActivityType != target.mActivityType) {
                if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Skipping " + task + ": mismatch activity type");
                continue;
            }
			
          	//判断Document相关
            final Intent taskIntent = task.intent;
            final Intent affinityIntent = task.affinityIntent;
            final boolean taskIsDocument;
            final Uri taskDocumentData;
            if (taskIntent != null && taskIntent.isDocument()) {
                taskIsDocument = true;
                taskDocumentData = taskIntent.getData();
            } else if (affinityIntent != null && affinityIntent.isDocument()) {
                taskIsDocument = true;
                taskDocumentData = affinityIntent.getData();
            } else {
                taskIsDocument = false;
                taskDocumentData = null;
            }

            if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Comparing existing cls="
                    + taskIntent.getComponent().flattenToShortString()
                    + "/aff=" + r.task.rootAffinity + " to new cls="
                    + intent.getComponent().flattenToShortString() + "/aff=" + info.taskAffinity);
            // TODO Refactor to remove duplications. Check if logic can be simplified.
            if (taskIntent != null && taskIntent.getComponent() != null &&
                    taskIntent.getComponent().compareTo(cls) == 0 &&
                    Objects.equals(documentData, taskDocumentData)) {
                if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Found matching class!");
                //dump();
                if (DEBUG_TASKS) Slog.d(TAG_TASKS,
                        "For Intent " + intent + " bringing to top: " + r.intent);
                result.r = r;
                result.matchedByRootAffinity = false;
                break;
            } else if (affinityIntent != null && affinityIntent.getComponent() != null &&
                    affinityIntent.getComponent().compareTo(cls) == 0 &&
                    Objects.equals(documentData, taskDocumentData)) {
                if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Found matching class!");
                //dump();
                if (DEBUG_TASKS) Slog.d(TAG_TASKS,
                        "For Intent " + intent + " bringing to top: " + r.intent);
                result.r = r;
                result.matchedByRootAffinity = false;
                break;
            } else if (!isDocument && !taskIsDocument
                    && result.r == null && task.canMatchRootAffinity()) {
              	//这个是判断是否匹配Task的关键
              	//通过Task的根部Affinity和目标Activity的taskAffinity去比较
                if (task.rootAffinity.equals(target.taskAffinity)) {
                    if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Found matching affinity candidate!");
                    // It is possible for multiple tasks to have the same root affinity especially
                    // if they are in separate stacks. We save off this candidate, but keep looking
                    // to see if there is a better candidate.
                    result.r = r;
                    result.matchedByRootAffinity = true;
                }
            } else if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Not a match: " + task);
        }
    }
```

``` java
//com.android.server.am.ActivityStarter
private void setTaskFromReuseOrCreateNewTask(TaskRecord taskToAffiliate) 
  		//获取是否有可重用的Task
        mTargetStack = computeStackFocus(mStartActivity, true, mLaunchBounds, mLaunchFlags,
                mOptions);

        if (mReuseTask == null) {
            final TaskRecord task = mTargetStack.createTaskRecord(
                    mSupervisor.getNextTaskIdForUserLocked(mStartActivity.userId),
                    mNewTaskInfo != null ? mNewTaskInfo : mStartActivity.info,
                    mNewTaskIntent != null ? mNewTaskIntent : mIntent,
                    mVoiceSession, mVoiceInteractor, !mLaunchTaskBehind /* toTop */);
          	//创建新Task并设置给目标Activity
            mStartActivity.setTask(task, taskToAffiliate);
            if (mLaunchBounds != null) {
                final int stackId = mTargetStack.mStackId;
                if (StackId.resizeStackWithLaunchBounds(stackId)) {
                    mService.resizeStack(
                            stackId, mLaunchBounds, true, !PRESERVE_WINDOWS, ANIMATE, -1);
                } else {
                    mStartActivity.task.updateOverrideConfiguration(mLaunchBounds);
                }
            }
            if (DEBUG_TASKS) Slog.v(TAG_TASKS,
                    "Starting new activity " +
                            mStartActivity + " in new task " + mStartActivity.task);
        } else {
            mStartActivity.setTask(mReuseTask, taskToAffiliate);
        }
    }

private ActivityStack computeStackFocus(ActivityRecord r, boolean newTask, Rect bounds,
            int launchFlags, ActivityOptions aOptions) {
        final TaskRecord task = r.task;
        if (!(r.isApplicationActivity() || (task != null && task.isApplicationTask()))) {
            return mSupervisor.mHomeStack;
        }
		//是否指定Task
        ActivityStack stack = getLaunchStack(r, launchFlags, task, aOptions);
        if (stack != null) {
            return stack;
        }

        if (task != null && task.stack != null) {
            stack = task.stack;
            if (stack.isOnHomeDisplay()) {
                if (mSupervisor.mFocusedStack != stack) {
                    if (DEBUG_FOCUS || DEBUG_STACK) Slog.d(TAG_FOCUS,
                            "computeStackFocus: Setting " + "focused stack to r=" + r
                                    + " task=" + task);
                } else {
                    if (DEBUG_FOCUS || DEBUG_STACK) Slog.d(TAG_FOCUS,
                            "computeStackFocus: Focused stack already="
                                    + mSupervisor.mFocusedStack);
                }
            }
            return stack;
        }

        final ActivityStackSupervisor.ActivityContainer container = r.mInitialActivityContainer;
        if (container != null) {
            // The first time put it on the desired stack, after this put on task stack.
            r.mInitialActivityContainer = null;
            return container.mStack;
        }

        // The fullscreen stack can contain any task regardless of if the task is resizeable
        // or not. So, we let the task go in the fullscreen task if it is the focus stack.
        // If the freeform or docked stack has focus, and the activity to be launched is resizeable,
        // we can also put it in the focused stack.
        final int focusedStackId = mSupervisor.mFocusedStack.mStackId;
        final boolean canUseFocusedStack = focusedStackId == FULLSCREEN_WORKSPACE_STACK_ID
                || (focusedStackId == DOCKED_STACK_ID && r.canGoInDockedStack())
                || (focusedStackId == FREEFORM_WORKSPACE_STACK_ID && r.isResizeableOrForced());
        if (canUseFocusedStack && (!newTask
                || mSupervisor.mFocusedStack.mActivityContainer.isEligibleForNewTasks())) {
            if (DEBUG_FOCUS || DEBUG_STACK) Slog.d(TAG_FOCUS,
                    "computeStackFocus: Have a focused stack=" + mSupervisor.mFocusedStack);
            return mSupervisor.mFocusedStack;
        }

        // We first try to put the task in the first dynamic stack.
        final ArrayList<ActivityStack> homeDisplayStacks = mSupervisor.mHomeStack.mStacks;
        for (int stackNdx = homeDisplayStacks.size() - 1; stackNdx >= 0; --stackNdx) {
            stack = homeDisplayStacks.get(stackNdx);
            if (!ActivityManager.StackId.isStaticStack(stack.mStackId)) {
                if (DEBUG_FOCUS || DEBUG_STACK) Slog.d(TAG_FOCUS,
                        "computeStackFocus: Setting focused stack=" + stack);
                return stack;
            }
        }

        // If there is no suitable dynamic stack then we figure out which static stack to use.
        final int stackId = task != null ? task.getLaunchStackId() :
                bounds != null ? FREEFORM_WORKSPACE_STACK_ID :
                        FULLSCREEN_WORKSPACE_STACK_ID;
        stack = mSupervisor.getStack(stackId, CREATE_IF_NEEDED, ON_TOP);
        if (DEBUG_FOCUS || DEBUG_STACK) Slog.d(TAG_FOCUS, "computeStackFocus: New stack r="
                + r + " stackId=" + stack.mStackId);
        return stack;
    }


private ActivityStack getLaunchStack(ActivityRecord r, int launchFlags, TaskRecord task,
            ActivityOptions aOptions) {

        // We are reusing a task, keep the stack!
        if (mReuseTask != null) {
            return mReuseTask.stack;
        }

        final int launchStackId =
                (aOptions != null) ? aOptions.getLaunchStackId() : INVALID_STACK_ID;

        if (isValidLaunchStackId(launchStackId, r)) {
            return mSupervisor.getStack(launchStackId, CREATE_IF_NEEDED, ON_TOP);
        } else if (launchStackId == DOCKED_STACK_ID) {
            // The preferred launch stack is the docked stack, but it isn't a valid launch stack
            // for this activity, so we put the activity in the fullscreen stack.
            return mSupervisor.getStack(FULLSCREEN_WORKSPACE_STACK_ID, CREATE_IF_NEEDED, ON_TOP);
        }

        if ((launchFlags & FLAG_ACTIVITY_LAUNCH_ADJACENT) == 0) {
            return null;
        }
        // Otherwise handle adjacent launch.

        // The parent activity doesn't want to launch the activity on top of itself, but
        // instead tries to put it onto other side in side-by-side mode.
        final ActivityStack parentStack = task != null ? task.stack
                : r.mInitialActivityContainer != null ? r.mInitialActivityContainer.mStack
                : mSupervisor.mFocusedStack;

        if (parentStack != mSupervisor.mFocusedStack) {
            // If task's parent stack is not focused - use it during adjacent launch.
            return parentStack;
        } else {
            if (mSupervisor.mFocusedStack != null && task == mSupervisor.mFocusedStack.topTask()) {
                // If task is already on top of focused stack - use it. We don't want to move the
                // existing focused task to adjacent stack, just deliver new intent in this case.
                return mSupervisor.mFocusedStack;
            }

            if (parentStack != null && parentStack.mStackId == DOCKED_STACK_ID) {
                // If parent was in docked stack, the natural place to launch another activity
                // will be fullscreen, so it can appear alongside the docked window.
                return mSupervisor.getStack(FULLSCREEN_WORKSPACE_STACK_ID, CREATE_IF_NEEDED,
                        ON_TOP);
            } else {
                // If the parent is not in the docked stack, we check if there is docked window
                // and if yes, we will launch into that stack. If not, we just put the new
                // activity into parent's stack, because we can't find a better place.
                final ActivityStack dockedStack = mSupervisor.getStack(DOCKED_STACK_ID);
                if (dockedStack != null
                        && dockedStack.getStackVisibilityLocked(r) == STACK_INVISIBLE) {
                    // There is a docked stack, but it isn't visible, so we can't launch into that.
                    return null;
                } else {
                    return dockedStack;
                }
            }
        }
    }
```

