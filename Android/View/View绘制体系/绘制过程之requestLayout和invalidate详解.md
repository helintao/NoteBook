# requestLayout
requestLayout是在View中定义的，并且在ViewGroup中没有重写该方法，它的注释是这样解释的：在需要刷新View的布局时调用这个函数，它会安排一个布局的传递。我们不应该在布局的过程中（isInLayout()）调用这个函数，如果当前正在布局，那么这一请求有可能在以下时刻被执行：当前布局结束、当前帧被绘制完或者下次布局发生时。
```java
public void requestLayout() {
    if (mMeasureCache != null) mMeasureCache.clear();
    if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == null) {
        // Only trigger request-during-layout logic if this is the view requesting it,
        // not the views in its parent hierarchy
        ViewRootImpl viewRoot = getViewRootImpl();
        if (viewRoot != null && viewRoot.isInLayout()) {
            if (!viewRoot.requestLayoutDuringLayout(this)) {
                return;
            }
        }
        mAttachInfo.mViewRequestingLayout = this;
    }
    mPrivateFlags |= PFLAG_FORCE_LAYOUT;
    mPrivateFlags |= PFLAG_INVALIDATED;
    if (mParent != null && !mParent.isLayoutRequested()) {
        mParent.requestLayout();
    }
    if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == this) {
        mAttachInfo.mViewRequestingLayout = null;
    }
}
```
在上面的代码当中，设置了两个标志位：PFLAG_FORCE_LAYOUT/PFLAG_INVALIDATED，除此之外最关键的一句话是：
```java
protected ViewParent mParent;
//....
mParent.requestLayout();
```
这个mParent存储的时候该View所对应的父节点，而当调用父节点的requestLayout()时，它又会调用它的父节点的requestLayout，就这样，以调用requestLayout的View为起始节点，一步步沿着View树传递上去。

整个View树的根节点是DecorView，DecorView中的mParent就是ViewRootImpl，而ViewRootImpl中的mView就是DecorView，所以，这一传递过程的终点就是ViewRootImpl的requestLayout方法
```java
@Override
public void requestLayout() {
    if (!mHandlingLayoutInLayoutRequest) {
        checkThread();
        mLayoutRequested = true;
        scheduleTraversals();
    }
}

void scheduleTraversals() {
    if (!mTraversalScheduled) {
        mTraversalScheduled = true;
        mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
        //该Runnable进行操作doTraversal.
        mChoreographer.postCallback(
                Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        if (!mUnbufferedInputDispatch) {
            scheduleConsumeBatchedInput();
        }
        notifyRendererOfFramePending();
        pokeDrawLockIfNeeded();
    }
}

final class TraversalRunnable implements Runnable {
    @Override
    public void run() {
        doTraversal();
    }
}
void doTraversal() {
    if (mTraversalScheduled) {
        mTraversalScheduled = false;
        mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
        if (mProfile) {
            Debug.startMethodTracing("ViewAncestor");
        }
        //这里最终会进行布局.
        performTraversals();
        if (mProfile) {
            Debug.stopMethodTracing();
            mProfile = false;
        }
    }
}
```
其中scheduleTraversals()中会执行一个mTraversalRunnable，该Runnable中最终会调用doTraversal，而doTraversal中执行的就是我们前面一直在谈到的performTraversals。


# invalidate
invalidate最终会调用到下面这个方法：
```java
void invalidateInternal(int l, int t, int r, int b, boolean invalidateCache,boolean fullInvalidate) {
    if (mGhostView != null) {
        mGhostView.invalidate(true);
        return;
    }
    if (skipInvalidate()) {
        return;
    }
    if ((mPrivateFlags & (PFLAG_DRAWN | PFLAG_HAS_BOUNDS)) == (PFLAG_DRAWN | PFLAG_HAS_BOUNDS)
            || (invalidateCache && (mPrivateFlags & PFLAG_DRAWING_CACHE_VALID) == PFLAG_DRAWING_CACHE_VALID)
            || (mPrivateFlags & PFLAG_INVALIDATED) != PFLAG_INVALIDATED
            || (fullInvalidate && isOpaque() != mLastIsOpaque)) {
        if (fullInvalidate) {
            mLastIsOpaque = isOpaque();
            mPrivateFlags &= ~PFLAG_DRAWN;
        }
        mPrivateFlags |= PFLAG_DIRTY;
        if (invalidateCache) {
            mPrivateFlags |= PFLAG_INVALIDATED;
            mPrivateFlags &= ~PFLAG_DRAWING_CACHE_VALID;
        }
        // Propagate the damage rectangle to the parent view.
        final AttachInfo ai = mAttachInfo;
        final ViewParent p = mParent;
        if (p != null && ai != null && l < r && t < b) {
            final Rect damage = ai.mTmpInvalRect;
            damage.set(l, t, r, b);
            p.invalidateChild(this, damage);
        }
        // Damage the entire projection receiver, if necessary.
        if (mBackground != null && mBackground.isProjected()) {
            final View receiver = getProjectionReceiver();
            if (receiver != null) {
                receiver.damageInParent();
            }
        }
    }
}
```
关键的一句是：p.invalidateChild(this, damage);

*在这里，p一定不为空并且它一定是一个ViewGroup。*

而ViewGroup当中的invalidateChildInParent会根据传入的区域来决定自己的绘制区域，和requestLayout类似，最终会调用ViewRootImpl的该方法：
```java
public ViewParent invalidateChildInParent(int[] location, Rect dirty){
    //···
}
```
