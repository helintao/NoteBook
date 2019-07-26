# LayoutAnimation
LayoutAnimation作用于ViewGroup，为ViewGroup指定一个动画，这样当它的子元素出场时都会具有这种动画效果。这种效果常常被用在ListView上，它的每一个item都以一定的动画的形式出现。

步骤如下：
1. 定义LayoutAnimation

```xml
<?xml version="1.0" encoding="utf-8"?>
<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
    android:delay="0.5"
    android:animationOrder="normal"
    android:animation="@anim/slide_in_left"/>
```
**android:delay**:表示子元素开始动画的时间延时

**android:animationOrder**：表示子元素动画的顺序，有三种选项：normal、reverse和random，其中normal表示顺序显示，reverse表示逆向显示，random表示随机显示。

**android:animation**：为子元素指定入场动画。

2. 为子元素指定具体的入场动画
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"
    android:shareInterpolator="true">

    <translate
        android:fromXDelta="500"
        android:toXDelta="0"/>

    <alpha
        android:fromAlpha="0.0"
        android:toAlpha="1.0"/>

</set>
```

3. 为ViewGroup指定android:layoutAnimation属性:android:layoutAnimaion="@anim/anim_layout"。对于ListView来说，这样ListView的item就具有出场动画，这种方式适用于所有ViewGroup
```xml
    ...
    <ListView
        android:id="@+id/next_lv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layoutAnimation="@anim/list_view_anim"></ListView>
    ...
```

*除了在XML中指定LayoutAnimation外，还可以通过LayoutAnimationController来实现*

# Activity的切换效果
Activity有默认的切换效果，但是这个效果我们是可以自定义的，主要用到overridePendingTransition(int enterAnim,int exitAnim)这个方法，**这个方法必须在startActivity(Intent)或者finish()之后被调用才能生效**，它的含义如下：

- enterAnim:Activity被打开时，所需的动画资源id。

- exitAnim:Activity被暂停时，所需的动画资源id。