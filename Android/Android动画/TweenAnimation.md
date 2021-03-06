# TweenAnimation（补间动画/View动画）
Tween Animation即补间动画，主要分为四种，分别是平移、缩放、旋转、透明度

## 语法

**在res/anim下创建xml文件**

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    <!--插值器-->
    android:interpolator="@[package:]anim/interpolator_resource"
    <!--动画结束后View是否停留在结束的位置-->
    android:fillAfter=["true" | "false"] 
    <!--重复的模式,默认为restart,即重头开始重新运行,reverse即从结束开始向前重新运行-->
    android:repeatMode="restart/reverse"
    <!--子元素是否共用此插值器-->
    android:shareInterpolator=["true" | "false"] >
    <alpha
        <!--开始透明度0.0（全透明）到1.0（完全不透明）-->
        android:fromAlpha="float"
        android:toAlpha="float"
        //动画执行时间
        android:duration="integer" />
    <scale 
        <!--其实X缩放大于0,1的时候表示不缩放，小于1缩小，大于1放大-->
        android:fromXScale="float"
        android:toXScale="float"
        android:fromYScale="float"
        android:toYScale="float"
        <!--缩放中心，也可以传fraction值-->
        android:pivotX="float"
        android:pivotY="float"
        android:duration="integer" />
    <translate
        <!--表示 x 的起始值-->
        android:fromXDelta="float/fraction"
        android:toXDelta="float"
        android:fromYDelta="float"
        android:toYDelta="float" 
        android:duration="integer" />
    <rotate
        <!--起始的旋转角度-->
        android:fromDegrees="float"
        android:toDegrees="float"
        android:pivotX="float"
        android:pivotY="float"
        android:duration="integer"  />
    <set>
        ...
    </set>
</set>
```

*set是一个动画集合，内部可以是多个动画的组合，也可以嵌套set，这里包含了动画实现的所有性,在上面的语法中我们需要注意的是平移的时候其实位置接受百分比数值:从-100到100的值，以“%”尾，表示百分比相对于自身;从-100到100的值，以“%p”结尾，表示百分比相对于父容器。例如平移开始位置自身中间则是50%,如平移开始位置在父容器的则是50%p.*

*例如有些人给我们的Activity会加一些从左边进右边出的动画，那么当我们打开ActivityActivity布局的fromXDelta值-100%p并将toXDelta为0%p,那么我们看到的效果就是从左边进入了。*

**EX**
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:shareInterpolator="true"
    android:interpolator="@android:anim/anticipate_overshoot_interpolator"
    android:fillAfter="true">

    <translate
        android:duration="1000"
        android:fromXDelta="50%p"
        android:fromYDelta="0%p"
        android:toXDelta="-50%"
        android:toYDelta="0%"/>

    <scale
        android:duration="1000"
        android:fromXScale="0.0"
        android:fromYScale="0.0"
        android:toXScale="1.0"
        android:toYScale="1.0"
        />

    <alpha
        android:duration="1000"
        android:fromAlpha="0.0"
        android:toAlpha="1.0" />
</set>
```
在代码中引用动画
```java
Animation animation= AnimationUtils.loadAnimation(context, R.anim.tween_animation);
imageView.startAnimation(animation);
```

# 进阶

View动画的四种变换效果对应着Animation的四个子类：TranslateAnimation、ScaleAnimation、RotateAnimation和AlphaAnimation。这四种动画既可以通过XML来定义，也可以通过代码来创建，对应View动画来说，建议采用xml来定义动画。这是因为xml格式的动画可读性更好。

## 监听过程
Animation的setAnimationListener可以添加过程监听
```java
    public interface AnimationListener {
        void onAnimationStart(Animation var1);
        void onAnimationEnd(Animation var1);
        void onAnimationRepeat(Animation var1);
    }
```

## 自定义动画

自定义动画是一件既简单又复杂的事情，说它简单，是因为派生一种新动画只需继承Animation这个抽象类，然后重写它的initialize和applyTransformation方法，在initialize方法中做一些初始化工作，在applyTransformation中进行相应的矩阵变换即可，很多时候需要采用Camera来简化矩阵变化的过程。说它复杂，在于矩阵变换。

