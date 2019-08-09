# 测量过程的信使:MeasureSpec
因为测量是一个从上到下的过程，而在这个过程当中，父容器有必要告诉子View它的一些绘制要求，那么这时候就需要依赖一个信使，来传递这个要求，它就是MeasureSpec.
MeasureSpec是一个32位的int类型，我们把它分为高2位和低30位。

其中高2位表示mode，它的取值为：

- UNSPECIFIED(0) : The parent has not imposed any constraint on the child. It can be whatever size it wants.(父节点没有对子节点施加任何约束。它可以是任意大小。)
- EXACTLY(1) : The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be.(父节点已经为子节点确定了确切的大小。不管子节点想要多大，他都会得到这些界限。)
- AT_MOST(2) : The child can be as large as it wants up to the specified size.(子元素可以任意大，直到指定的大小。)

低30位表示具体的size。

```
MeasureSpec是父容器传递给子View的宽高要求，并不是说它传递的size是多大，子View最终就是多大，它是根据父容器的MeasureSpec和子View的LayoutParams共同计算出来的。
```
从源码中就可以分析出：
```java
protected void measureChildWithMargins(View child,
        int parentWidthMeasureSpec, int widthUsed,
        int parentHeightMeasureSpec, int heightUsed) {
    final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                    + widthUsed, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                    + heightUsed, lp.height);
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}

public static int getChildMeasureSpec(int spec, int padding, int childDimension {
    int specMode = MeasureSpec.getMode(spec);
    int specSize = MeasureSpec.getSize(spec);
    int size = Math.max(0, specSize - padding);
    int resultSize = 0;
    int resultMode = 0;
    switch (specMode) {
    // Parent has imposed an exact size on us
    case MeasureSpec.EXACTLY:
        if (childDimension >= 0) {
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size. So be it.
            resultSize = size;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;
    // Parent has imposed a maximum size on us
    case MeasureSpec.AT_MOST:
        if (childDimension >= 0) {
            // Child wants a specific size... so be it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size, but our size is not fixed.
            // Constrain child to not be bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            resultSize = size;
            resultMode = MeasureSpec.AT_MOST;
        }
        break;
    // Parent asked to see how big we want to be
    case MeasureSpec.UNSPECIFIED:
        if (childDimension >= 0) {
            // Child wants a specific size... let him have it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size... find out how big it should
            // be
            resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
            resultMode = MeasureSpec.UNSPECIFIED;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size.... find out how
            // big it should be
            resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
            resultMode = MeasureSpec.UNSPECIFIED;
        }
        break;
    }
    //noinspection ResourceType
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
```
在调用getChildMeasureSpec之前，需要考虑parent和child之间的间距，这包括parent的padding和child的margin，因此，参与传递给child的MeasureSpec的参数要考虑这么几方面：

- 父容器的measureSpec和padding

- 子View的height和widht以及margin

## 分析getChildMeasureSpec
- **父容器的mode为EXACTLY**

    这种情况下说明父容器的大小已经确定了，就是固定的值。

    - 子View指定了大小

        那么子View的mode就是EXACTLY，size就是布局里面的值

    - 子View为MATCH_PARENT

        子View希望和父容器一样大，因为父容器的大小是确定的，所以子View的大小也是确定的，size就是父容器measureSpec的size - 父容器的padding - 子View的margin

    - 子View为WRAP_CONTENT

        子容器只要求能够包裹自己的内容，但是这时候它又不知道它所包裹的内容到底是多大，那么这时候它就指定自己的大小就不能超过父容器的大小，所以mode为AT_MOST，size和上面类似。

- **父容器的mode为AT_MOST**

    在这种情况下，父容器说明了自己最多不能超过多大，数值在measureSpec的size当中：

    - 子View指定大小

        同上分析。

    - 子View为MATCH_PARENT

        子View希望和父容器一样大，而此时父容器只知道自己不能超过多大，因此子View也就只能知道自己不能超过多大，所以它的mode为AT_MOST，size就是父容器measureSpec的size - 父容器的padding - 子View的margin。

    - 子View为WRAP_CONTENT

        子容器只要求能够包裹自己的内容，但是这时候它又不知道它所包裹的内容到底是多大，这时候虽然父容器没有指定大小，但是它指定了最多不能超过多少，这时候子View也不能超过这个值，所以mode为AT_MOST，size的计算和上面类似。

- ** 父容器的mode为UNSPECIFIED**

    - 子View指定大小

        同上分析。

    - 子View为MATCH_PARENT

        子View希望和父容器一样大，但是这时候父容器并没有约束，所以子View也是没有约束的，所以它的mode也为UNSPECIFIED，size的计算和之前一致。

    - 子View为WRAP_CONTENT

        子View不知道它包裹的内容多大，并且父容器是没有约束的，那么也只能为UNSPECIFIED了，size的计算和之前一致。

# 测量过程的起点performMeasures()
从View树的根节点到叶节点的整个测量的过程。整个测量的起点是在ViewRootImpl的performMeasure()当中:
```java
private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
    if (mView == null) {
        return;
    }
    Trace.traceBegin(Trace.TRACE_TAG_VIEW, "measure");
    try {
        mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_VIEW);
    }
}
``` 
上面的mView也就是我们在setContentView当中渲染出来的mDecorView，也就是说它是整个View树的根节点，因为mDecorView是一个FrameLayout，所以它调用的是FrameLayout的measure方法。

那么这整个从根节点遍历完整个View树的过程是怎么实现的呢？

它其实就是依赖于measure和onMeasure：

- 对于View，measure是在它里面定义的，而且它是一个final方法，因此它的所有子类都没有办法重写该方法，在该方法当中，会调用onMeasure来设置最终测量的结果，对于View来说，它只是简单的取出父容器传进来的要求来设置，并没有复杂的逻辑。

- 对于ViewGroup，由于它是View的子类，因此它不可能重写measure方法，并且它也没有重写onMeasure方法。

- 对于继承于View的控件，例如TextView，它会重写onMeasure，与View#onMeasure不同的是，它会考虑更多的情况来决定最终的测量结果。

- 对于继承于ViewGroup的控件，例如FrameLayout，它同样会重写onMeasure方法，与继承于View的控件不同的是，由于ViewGroup可能会有子View，因此它在设置自己最终的测量结果之前，还有一个重要的任务：调用子View的measure方法，来对子View进行测量，并根据子View的结果来决定自己的大小。

因此，整个从上到下的测量，其实就是一个View树节点的遍历过程，每个节点的onMeasure返回时，就标志它的测量结束了，而这整个的过程是以View中measure方法为纽带的：

- 整个过程的起点是mDecorView这个根节点的measure方法，也就是performMeasure中的那句话。

- 如果节点有子节点，也就是说它是继承于ViewGroup的控件，那么在它的onMeasure方法中，它并不会直接调用子节点的onMeasure方法，而是通过调用子节点measure方法，由于子节点不可能重写View#measure方法，因此它最终是通过View#measure来调用子节点重写的onMeasure来进行测量，子节点再在其中进行响应的逻辑处理。

- 如果节点没有子节点，那么当它的onMeausre方法被调用时，它需要设置好自己的测量结果就行了。

对于measure和onMeasure的区别，我们可以用一句简单的话来总结一下：measure负责进行测量的传递，onMeasure负责测量的具体实现。

# 测量过程的终点onMeasure当中的setMeasuredDimension
上面我们讲到设置的测量结果，其实测量过程的最终目的是：通过调用setMeasuredDimension方法来给mMeasureHeight和mMeasureWidth赋值。

只要上面这个过程完成了，那么该ViewGroup/View/及其实现类的测量也就结束了，而setMeasuredDimension必须在onMeasure当中调用，否则会抛出异常，所以我们观察所有继承于ViewGroup/View的控件，都会发现它们最后都是调用上面说的那个方法。

## View的onMeasure
```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}

public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);
    switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY:
        result = specSize;
        break;
    }
    return result;
}

```
父容器传递进来measureSpec中的mode来给这两个变量赋值：

- 如果mode为UNSPECIFIED，那么说明父容器并不指望多个，因此子View根据自己的背景或者minHeight/minWidth属性来给自己赋值。
- 如果是AT_MOST或者EXACTLY，那么就把它设置为父容器指定的size。

## ViewGroup的onMeasure
由于ViewGroup的目的是为了容纳各子View，但是它并不确定子View应当如何排列，也就不知道该如何测量自己，因此它的onMeasure是没有任何意义的，所以并没有重写，而是应当由继承于它的控件来重写该方法。

## 继承于ViewGroup控件的onMeasure
整个的onMeasure其实分为三步：

- 遍历所有子View，调用measureChildWithMargins进行第一次子View的测量，它最终也是调用子View的measure方法。
- 根据第一步的结果，调用setMeasuredDimension来设置自己的测量结果。
- 遍历所有子View，根据第二步的结果，调用child.measure进行第二次的测量。

结论：父容器和子View的关联是通过measure进行关联的。
同时我们也可以有一个新的结论，对于View树的某个节点，它的测量结果有可能并不是一次决定的，这是由于父容器可能需要依赖于子View的测量结果，而父容器的结果又可能会影响子View，但是，我们需要保证这个过程不是无限调用的。
