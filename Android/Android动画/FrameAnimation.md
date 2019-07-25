# FrameAnimation(帧动画)
帧动画是最容易实现的一种动画，这种动画更多的依赖于完善的UI资源，他的原理就是将一张张单独的图片连贯的进行播放，从而在视觉上产生一种动画的效果；有点类似于某些软件制作gif动画的方式。

## 语法

**在res/drawable下创建xml文件**

- drawable文件夹下创建一个xml文件
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <animation-list xmlns:android="http://schemas.android.com/apk/res/android"
        android:oneshot=["true" | "false"] >
        <item
            android:drawable="@[package:]drawable/drawable_resource_name"
            android:duration="integer" />
    </animation-list>
    ```
    **oneshot默认为false，该动画会一直循环执行，当我们设置true后则播放到最后一帧时动画停止。**

    **EX:**
    ```xml
    <animation-list xmlns:android="http://schemas.android.com/apk/res/android"
        android:oneshot="true">
        <item
            android:drawable="@drawable/xhr_1"
            android:duration="200" />
        <item
            android:drawable="@drawable/xhr_2"
            android:duration="100" />
        <item
            android:drawable="@drawable/xhr_3"
            android:duration="100" />
        <item
            android:drawable="@drawable/xhr_4"
            android:duration="100" />
    </animation-list>
    ```
    //在代码中引用动画
    ```java
    imageView.setImageResource(R.drawable.frame_animation);
    animationDrawable= (AnimationDrawable) imageView.getDrawable();
    animationDrawable.start();
    ```

- 代码实现

    ```java
    AnimationDrawable animationDrawable = new AnimationDrawable();
    for(int i=1;i<=35;i++){
        int id = context.getResources().getIdentifier(
                "xhr_"+i,"drawable",context.getPackageName());
        Drawable drawable=context.getResources().getDrawable(id);
        animationDrawable.addFrame(drawable,50);
    }
    animationDrawable.setOneShot(false);
    imageView.setBackgroundDrawable(animationDrawable);
    animationDrawable.start();
    ```