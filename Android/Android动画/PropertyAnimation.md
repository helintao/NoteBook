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

