* `launchMode`和`flags`同时存在时，`flags`覆盖`launchMode`。
* 当flags指定了`NEW_TASK`时，`taskAffinity`必须指定与默认值(包名)不同的值，才会新建Task(如果目标Activity实例不存在时)。
* 当flags指定了`NEW_TASK`时，且目标Activity实例存在时，会将目标Task(目标Actvity所在的Task)前置，如果前台Task不为目标Task的话。
* `CLEAR_TASK`只能和`NEW_TASK`配合使用，将目标task(根据`taskAffinity`判断)清空，再将目标Activity新实例压入。
* `ClEAR_TOP`的作用是如果当前Task存在目标Activity实例，则该实例前面的Activity推出，然后创建新目标Activity实例(**即使目标Activity已经位于栈顶，也会destroy，重新创建实例**)位于栈顶。
  * 当和`NEW_TASK`配合使用时，则会根据`taskAffinity`寻找目标Task，如果存在目标Activity实例时，则将目标栈中目标Activity实例前面的Activity推出，让创建新目标Activity实例位于栈顶(**即使目标Activity已经位于栈顶**)。否则，则创建目标Activity实例并压入目标Tsk(如果目标Task不存在则创建Task，根据`taskAffinity`寻找目标Task)。
  * 当和`SINGLE_TOP`配合使用时，如果当前Task存在目标Activity实例时，则该实例前面的Activity推出，让该实例位于栈顶(**注意区别和`CLEAR_TOP`单独使用，关键在于有没有创建新实例**)。
  * 当和`NEW_TASK`，`SINGLE_TOP`一起配合使用时，如果目标Activity实例存在时，则将目标Task中位于目标Activity前面的Activity推出，让该实例(不重新创建实例)位于栈顶。


```java
final int startActivityLocked(...) {
        ...
        final int launchFlags = intent.getFlags();

        if ((launchFlags & Intent.FLAG_ACTIVITY_FORWARD_RESULT) != 0 && sourceRecord != null) {
          	//当设置了这个标志后，如果起始Activity设置了requestCode，那么跳转到新的Activity中setResult，依然会返回到起始Activity中。
            ...
        }

        if (err == ActivityManager.START_SUCCESS && intent.getComponent() == null) {
            // We couldn't find a class that can handle the given Intent.
            // That's the end of that!
          	//没有找到可以处理Intent的类
            err = ActivityManager.START_INTENT_NOT_RESOLVED;
        }

        if (err == ActivityManager.START_SUCCESS && aInfo == null) {
            // We couldn't find the specific class specified in the Intent.
            // Also the end of the line.
          	//没有找到指定的类
            err = ActivityManager.START_CLASS_NOT_FOUND;
        }

        if (err == ActivityManager.START_SUCCESS && sourceRecord != null
                && sourceRecord.task.voiceSession != null) {
          	//当前为语音交互
            ...
        }

        if (err == ActivityManager.START_SUCCESS && voiceSession != null) {
          	//语音交互
            ...
        }

       ...

        try {
            mService.mWindowManager.deferSurfaceLayout();
          	//关键语句
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
 private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask) {

        setInitialState(r, options, inTask, doResume, startFlags, sourceRecord, voiceSession,
                voiceInteractor);

        computeLaunchingTaskFlags();

        computeSourceStack();

        mIntent.setFlags(mLaunchFlags);

        mReusedActivity = getReusableIntentActivity();

        final int preferredLaunchStackId =
                (mOptions != null) ? mOptions.getLaunchStackId() : INVALID_STACK_ID;

        if (mReusedActivity != null) {
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

