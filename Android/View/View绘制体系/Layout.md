# 布局的起点performLayout
整个布局的起点是在ViewRootImpl的performLayout当中：
```java
 private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,int desiredWindowHeight) {
    final View host = mView;
    \\···
    host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
    \\···
 }
```
布局过程也有两个方法layout和onLayout

## layout和onLayout
### **对于View**

    layout方法是public的，和measure不同的是，它不是final的，也就是说，继承于View的控件可以重写layout方法，但是我们一般不这么做，因为在它的layout方法中又调用了onLayout，所以继承于View的控件一般是通过重写onLayout来实现一些逻辑。

**layout**
```java
public void layout(int l, int t, int r, int b) {
    //....
    boolean changed = isLayoutModeOptical(mParent) ?
            setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);
    if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == FLAG_LAYOUT_REQUIRED) {
        onLayout(changed, l, t, r, b);
    }
    //...
}
```

**onLayout**
```java
/**
 * Called from layout when this view should
 * assign a size and position to each of its children.
 *
 * Derived classes with children should override
 * this method and call layout on each of
 * their children.
 * @param changed This is a new size or position for this view
 * @param left Left position, relative to parent
 * @param top Top position, relative to parent
 * @param right Right position, relative to parent
 * @param bottom Bottom position, relative to parent
 */
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
}
```

### **对于ViewGroup**
它重写了View中的layout，并把它设为final，也就是说继承于ViewGroup的控件，不能重写layout，在layout方法中，又会调用super.layout，也就是View的layout。
```java
@Override
public final void layout(int l, int t, int r, int b) {
    if (!mSuppressLayout && (mTransition == null || ansition.isChangingLayout())) {
        if (mTransition != null) {
            mTransition.layoutChange(this);
        }
        super.layout(l, t, r, b);
    } else {
        // record the fact that we noop'd it; request ut when transition finishes
        mLayoutCalledWhileSuppressed = true;
    }
}
```

onLayout方法重写View中的onLayout方法，并把它声明成了abstract，也就是说，所有继承于ViewGroup的控件，都必须实现onLayout方法。
```java
@Override
protected abstract void onLayout(boolean changed,
        int l, int t, int r, int b);
```

### **对于继承于View的控件**
例如TextView，我们一般不会重写layout，而是在onLayout中进行简单的处理。

### **对于继承于ViewGroup的控件**
例如LinearLayout，由于ViewGroup的作用是为了包裹子View，而每个控件由于作用不同，布局的方法自然也不同，这也是为了安卓要求每个继承于ViewGroup的控件都必须实现onLayout方法的原因。

因为ViewGroup的layout方法不可以重写，因此，当我们通过父容器调用一个继承于ViewGroup的控件的layout方法时，它最终会回调到该控件的onLayout方法。

## 布局onLayout(boolean changed, int l, int t, int r, int b)参数说明
这个矩形的四个坐标是相对于父容器的坐标值。

## 布局的遍历过程
当从父容器到子View传递的过程中，我们不直接调用onLayout，而是调用layout。

onMeasure在测量过程中负责两件事：它自己的测量和它的子View的测量，而onLayout不同：它并不负责自己的布局，这是由它的父容器决定的，它仅仅负责自己的下一级子View的布局。

FrameLayout的onLayout方法：
```java
protected void onLayout(boolean changed, int left, int top int right, int bottom) {
    layoutChildren(left, top, right, bottom, false /* no force left gravity */);
}

void layoutChildren(int left, int top, int right, intbottom, boolean forceLeftGravity) {
    final int count = getChildCount();
    final int parentLeft = getPaddingLeftWithForeground();
    final int parentRight = right - left - getPaddingRightWithForeground();
    final int parentTop = getPaddingTopWithForeground();
    final int parentBottom = bottom - top - getPaddingBottomWithForeground();
    for (int i = 0; i < count; i++) {
        final View child = getChildAt(i);
        if (child.getVisibility() != GONE) {
            final LayoutParams lp = (LayoutParams) child.getLayoutParams();
            final int width = child.getMeasuredWidth();
            final int height = child.getMeasuredHeight();
            int childLeft;
            int childTop;
            int gravity = lp.gravity;
            if (gravity == -1) {
                gravity = DEFAULT_CHILD_GRAVITY;
            }
            final int layoutDirection = getLayoutDirection();
            final int absoluteGravity = Gravity.getAbsoluteGravity(gravity, layoutDirection);
            final int verticalGravity = gravity & Gravity.VERTICAL_GRAVITY_MASK;
            switch (absoluteGravity & Gravity.HORIZONTAL_GRAVITY_MASK) {
                case Gravity.CENTER_HORIZONTAL:
                    childLeft = parentLeft + (parentRight - parentLeft - width) / 2 +
                    lp.leftMargin - lp.rightMargin;
                    break;
                case Gravity.RIGHT:
                    if (!forceLeftGravity) {
                        childLeft = parentRight - width - lp.rightMargin;
                        break;
                    }
                case Gravity.LEFT:
                default:
                    childLeft = parentLeft + lp.leftMargin;
            }
            switch (verticalGravity) {
                case Gravity.TOP:
                    childTop = parentTop + lp.topMargin;
                    break;
                case Gravity.CENTER_VERTICAL:
                    childTop = parentTop + (parentBottom - parentTop - height) / 2 +
                    lp.topMargin - lp.bottomMargin;
                    break;
                case Gravity.BOTTOM:
                    childTop = parentBottom - height - lp.bottomMargin;
                    break;
                default:
                    childTop = parentTop + lp.topMargin;
            }
            child.layout(childLeft, childTop, childLeft + width, childTop + height);
        }
    }
}
```