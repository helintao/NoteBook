# PropertyAnimation(属性动画)
属性动画是3.0之后引入的，在View动画中虽然我们看到了我们的控件位置发生发生变化，比如Button虽然位置变化了，但是点击响应区域还在原来的位置。而属性动画就可以解决这种问题。它可以作用于View的属性。

## 语法

**在res/animator下创建xml文件**

```xml
<set
//执行的顺序together同时执行，sequentially连续执行
  android:ordering=["together" | "sequentially"]>

    <objectAnimator
        //属性动画名字，例如"alpha"或者"backgroundColor"等
        android:propertyName="string"
        android:duration="int"
        //属性的开始值
        android:valueFrom="float | int | color"
        android:valueTo="float | int | color"
        //该动画开始的延迟时间
        android:startOffset="int"
        //动画重复次数，-1表示一直循环，1表示循环一次也就是播放两次，默认0，播放一次
        android:repeatCount="int"
        //repeatCount设置-1时才会有效果
        android:repeatMode=["repeat" | "reverse"]
        //如果值是颜色，则不用使用此属性
        android:valueType=["intType" | "floatType"]/>

    <animator
        android:duration="int"
        android:valueFrom="float | int | color"
        android:valueTo="float | int | color"
        android:startOffset="int"
        android:repeatCount="int"
        android:repeatMode=["repeat" | "reverse"]
        android:valueType=["intType" | "floatType"]/>

    <set>
        ...
    </set>
</set>
```

        <set>标签对应AnimatorSet,<animator>标签对应ValueAnimator,<objectAnimator>标签对应ObjectAnimator

*下面列出了常见的属性名字，另外需要注意的是，使用属性动画时，必须有相应属性的set/get方法，否则属性动画没有效果的。*

- translationX和translationY:控制View距离左边和顶部的距离的增加值。是一个相对值。相对于自身位置的具体。

- rotation、rotationX和rotationY:rotation是控制View围绕其支点进行旋转。rotationX和 rotationY分别是围绕X轴和Y轴旋转。

- scaleX和scaleY:控制View的缩放。

- pivotX和pivotY:控制View的支点位置，进行旋转和缩放，默认是View的中点。它们都是float值，0表示View的最左边和最顶端，1表示最右端和最下端。

- alpha:控制View的透明度。

- x和y:控制View在布局容器中距离左边和顶部的距离。是一个绝对值。

**EX:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:ordering="sequentially">
    <objectAnimator
        android:duration="500"
        android:propertyName="rotation"
        android:valueFrom="0"
        android:valueTo="180" />
    <objectAnimator
        android:duration="500"
        android:propertyName="alpha"
        android:valueFrom="1.0f"
        android:valueTo="0.5f" />
</set>
```
```java
//在代码中引用
AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(this,
                R.animator.property_animator);
set.setTarget(imageView);
set.start();
```

**在java代码中直接使用**
```java
ObjectAnimator.ofFloat(imageView,"rotation",0,180,90,180)
                .setDuration(2000).start();
```

## 如果我们想同时作用几个属性
有两种实现方式分别是类PropertyValuesHolder和AnimatorSet
```java
private void startPropertyAnimation3() {
    PropertyValuesHolder translationX = PropertyValuesHolder
            .ofFloat("translationX", -200, -100, 100, 200, 300);
    PropertyValuesHolder scaleX = PropertyValuesHolder
            .ofFloat("scaleX", 1.0f, 2.0f);
    PropertyValuesHolder rotate = PropertyValuesHolder
            .ofFloat("rotation", 0f, 360f);
    PropertyValuesHolder rotationX = PropertyValuesHolder
            .ofFloat("rotationX", 0f, 180f);
    ObjectAnimator together = ObjectAnimator
            .ofPropertyValuesHolder(imageView, translationX, rotate, scaleX, rotationX);
    together.setDuration(3000);
    together.start();
}

//或者使用AnimatorSet,此方法使用的是按顺序播放。
private void startPropertyAnimation4() {
    ObjectAnimator translationX = ObjectAnimator
            .ofFloat(imageView, "translationX", -200, -100, 100, 200, 300);
    ObjectAnimator scaleX = ObjectAnimator
            .ofFloat(imageView, "scaleX", 1.0f, 2.0f)
            .setDuration(1000);
    ObjectAnimator rotation = ObjectAnimator
            .ofFloat(imageView, "rotation", 0f, 360f)
            .setDuration(1000);
    ObjectAnimator rotationX = ObjectAnimator
            .ofFloat(imageView, "rotationX", 0f, 180f)
            .setDuration(1000);
    AnimatorSet set = new AnimatorSet();
    set.playSequentially(translationX, scaleX, rotation, rotationX);
    set.setDuration(4000);
    set.start();
}
```

## 属性动画的监听器

### AnimatorListener

```java
public interface AnimatorListener {
    default void onAnimationStart(Animator animation, boolean isReverse) {
        throw new RuntimeException("Stub!");
    }
    default void onAnimationEnd(Animator animation, boolean isReverse) {
        throw new RuntimeException("Stub!");
    }
    void onAnimationStart(Animator var1);
    void onAnimationEnd(Animator var1);
    void onAnimationCancel(Animator var1);
    void onAnimationRepeat(Animator var1);
}
```
从AnimatorListener的定义可以看出，它可以监听动画的开始、结束、取消以及重复播放。同时为了方便开发，系统还提供了AnimatorListenerAdapter这个类，它是AnimatorListener的适配器类，我们可以有选择的实现上面的4个方法。

### AnimatorUpdateListener

```java
public interface AnimatorUpdateListener {
    void onAnimationUpdate(ValueAnimator var1);
}
```
AnimatorUpdateListener比较特殊，它会监听整个动画过程，动画是由许多帧组成的，每播放一帧，onAnimationUpdate就会被调用一次，利用这个特性，我们可以做一些特殊的事情。

## 属性动画原理：
属性动画要求动画作用的对象提供该属性的get和set方法，属性动画根据外界传递的该属性的初始值和最终值，以动画的效果多次去调用set方法，每次传递给set方法的值都不一样，切确来说是随着时间的推移，所传递的值越来越接近最终值。

总结：我们对object的属性abc做动画，如果想让动画生效，要同时满足两个条件：
1. object必须要提供setAbc方法，如果动画的时候没有传递初始值，那么还要提供getAbc方法，因为系统要去取abc属性的初始值(如果这条不满足，程序直接Crash)。

2. object的setAbc对属性abc所做的改变必须能够通过某种方法反映出来，比如会带来UI的改变之类的(如果这条不满足，动画无效果但不会Crash)。

## 对任意属性做动画

对于属性无法做动画，官方有三种解决方法：

- 给你的对象加上get和set方法，如果你有权限的话

这个方法最简单，但是往往是不可行的，比如无法给Button加上一个合乎要求的setWidth方法。

- 用一个类来包装原始对象，间接为其提供get和set方法

**EX:**
```java
private void performAnimate(){
    ViewWrapper wrapper = new ViewWrapper(mButton);
    ObjectAnimator.ofInt(wrapper,"width",500).setDuration(5000).start();
}

private static class ViewWrapper{
    private View mTarget;

    public ViewWrapper(View target){
        mTarget = target;
    }

    public int getWidth(){
        return mTarget.getLayoutParams.width;
    }

    public void set Width(int width){
        mTarget.getLayoutParams().width = width;
        mTarget.requestLayout();
    }
}
```

- 采用ValueAnimator，监听动画过程，自己实现属性的改变


