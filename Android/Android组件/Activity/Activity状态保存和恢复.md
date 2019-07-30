在开发过程中，不可避免地会遇到Activity被回收的场景， Activity被回收有两种情况：主动和被动。

- 当Activity是被主动回收时，例如按下了Back键，那么这时候是无法恢复的，因为系统认为你已经不再需要它了

- 在被动回收的情况下，虽然这个Activity的实例已经被销毁了，但是系统在新建一个Activity实例的时候，会带上先前被回收Activity的信息，这些信息是被存储在Bundle的键值对，这里面有些是系统帮我们读写的，例如Activity当中View的状态，这部分的信息并不需要我们担心。（为了系统能帮我们恢复状态，必须要为每个View都指定一个id），如果我们希望保存更多临时的信息，而这些信息又没有必要写入到持久化的存储当中，这时候我们应该重写onSaveInstanceState方法，在该回调当中会传入一个Bundle对象，只需要把保存的信息放到里面，之后再在onRestoreInstanceState和onCreate方法中把它读取出来就可以了，这里面有两点需要注意的：

    - onRestoreInstance只有在Bundle不为空时才会回调，而在onCreate方法当中是通过判空操作来判断是否有需要恢复的状态的。

    - 在重写onRestoreInstanceState方法时，应当先调用super方法，这样由系统负责保存的部分才能够恢复。

# 对于View的状态，是怎么做到自动恢复的
由于文档要求我们必须调用super方法，那么就可以知道保存View状态的代码入口必然在Activity当中，我们看下Activity的默认实现：
```java
<!-- Activity.java -->
protected void onSaveInstanceState(Bundle outState) {
    outState.putBundle(WINDOW_HIERARCHY_TAG, mWindow.saveHierarchyState());
    outState.putInt(LAST_AUTOFILL_ID, mLastAutofillId);
    Parcelable p = mFragments.saveAllState();
    if (p != null) {
        outState.putParcelable(FRAGMENTS_TAG, p);
    }
    if (mAutoFillResetNeeded) {
        outState.putBoolean(AUTOFILL_RESET_NEEDED, true);
        getAutofillManager().onSaveInstanceState(outState);
    }
    getApplication().dispatchActivitySaveInstanceState(this, outState);
}

//看到mWindow，自然就想到了PhoneWindow.java
<!-- PhoneWindow.java -->
public Bundle saveHierarchyState() {
     mContentParent.saveHierarchyState(states); //这里是保存View状态的地方。  
     outState.putInt(FOCUSED_ID_TAG, focusedView.getId()); //保存Focus。
     outState.putSparseParcelableArray(PANELS_TAG, panelStates); //保存Panel状态。
}

<!-- View.java -->
public void saveHierarchyState(SparseArray<Parcelable> container) {
    dispatchSaveInstanceState(container);
}

protected void dispatchSaveInstanceState(SparseArray<Parcelable> container) {
    if (mID != NO_ID && (mViewFlags & SAVE_DISABLED_MASK) == 0) {
        mPrivateFlags &= ~PFLAG_SAVE_STATE_CALLED;
        Parcelable state = onSaveInstanceState();
        if ((mPrivateFlags & PFLAG_SAVE_STATE_CALLED) == 0) {
            throw new IllegalStateException(
                    "Derived class did not call super.onSaveInstanceState()");
        }
        if (state != null) {
            // Log.i("View", "Freezing #" + Integer.toHexString(mID)
            // + ": " + state);
            container.put(mID, state);
        }
    }
}
```

整个保存View状态的流程如下：

- 调用Activity的onSaveInstanceState方法,该方法又调用mWindow.saveHierarchyState，把返回的结果保存到WINDOW_HIERARCHY_TAG这个Key对应的Value中

- mWindow的实现类PhoneWindow当中：调用根布局的saveHierarchyState方法，这里面会从根布局按树形结构遍历，调用每个ViewGroup/View的onSaveInstanceState

- 保存FoucusView

我们也可以得到结论，保存的前提有两个:
- View的子类必须实现了onSaveInstanceState
- 它必须要有一个ID，这个ID作为Bundle的key，这也为我们实现自定义 View 时，需要保存状态提供了思路。

# onSaveInstanceState调用时机
- 如果是honeycomb之后的版本，那么在performPauseActivity时是不会保存的，而对于honeycomb之前的版本，会在回调onPause()之前保存。

- 如果在performPauseActivity时没有保存，那么在执行performStopActivityInner时r.state为空并且是从handleStopActivity过来的，那么会在onPause()和onStop()之间保存状态，其它情况下不会保存状态。

- 在ReLaunchAcitivity时，也会保存状态。

# onRestoreInstanceState调用时机
在Activity的生命周期解析中分析onStart()方法的时候我们已经知道，它是在onStart()和onResume()之间被调用的。
