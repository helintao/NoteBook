# 在Fragment中使用

**EX**
```java
FragmentTransaction transaction = fragmentManager.beginTransaction();
transaction.setCustomAnimations(R.anim.slide_in_left, R.anim.slide_out_right);
transaction.replace(R.id.fragment_container, fragment, fragmentTag);
transaction.commit();
```

#Activity中使用
有两种方式：
1. 在Activity中提供了overridePendingTransition(int enterAnim, int exitAnim) 方法，该方法接收两个参数，第一个参数是Activity进入时的动画，第二个参数是Activity退出时的动画。该方法一般写在startActivity()后和finish（）后，如果我们想打开或者退出不显示动画，可将参数设置为0。

2. 在主题中设置动画。
```xml
    <style name="CustomeActivityStyle" parent="AppTheme">
        <item name="android:windowAnimationStyle">@style/AnimationStyle</item>
    </style>
    <style name="AnimationStyle">
        <item name="android:activityCloseEnterAnimation">@anim/slide_in_left</item>
        <item name="android:activityCloseExitAnimation">@anim/slide_out_right</item>
        <item name="android:activityOpenEnterAnimation">@anim/slide_in_left</item>
        <item name="android:activityOpenExitAnimation">@anim/slide_out_right</item>
    </style>
```

*为什么是设置了四种，假如有Activity1和Activity2。当我们在Activity1中跳转到Activity2时，Activity1在页面上消失是执行的:activityOpenExitAnimation动画，Activity2出现在屏幕上执行的动画是activityOpenEnterAnimation。当Activity2  finish返回Activity1时，Activity2执行的动画是activityCloseExitAnimation，Activity1显示在屏幕执行的是activityCloseEnterAnimation。创建上面主题后我们需要将该主题应用到我们的Activty中就可以了。*

*同理Fragment也可相应设置，如activityCloseEnterAnimation改为fragmentCloseEnterAnimation即可。除此之外我们也可以使用windowAnimation，它包括 windowEnterAnimation 和 windowExitAnimation。注意的一点就是WindowAnimation的控制权大于Activity/Fragment Animation的控制权。*