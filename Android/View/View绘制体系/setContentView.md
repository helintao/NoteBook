在Activity当中，我们一般都会调用setContentView方法来初始化布局。

# 与ContentView相关的方法

```java
@Override
public void setContentView(@LayoutRes int layoutResID) {
    getDelegate().setContentView(layoutResID);
}
@Override
public void setContentView(View view) {
    getDelegate().setContentView(view);
}
@Override
public void setContentView(View view, ViewGroup.LayoutParams params) {
    getDelegate().setContentView(view, params);
}
@Override
public void addContentView(View view, ViewGroup.LayoutParams params) {
    getDelegate().addContentView(view, params);
}
```

这4个方法的用途：

- 第一种：渲染layoutResId对应的布局，并将它添加到activity的顶级View中。
- 第二种：将View添加到activity的布局当中，它的默认宽高都是ViewGroup.LayoutParams#MATCH_PARENT。
- 第三种：和上面相同，但是指定了LayoutParams。
- 第四种：将内容添加进去，并且必须指定LayoutParams，已经存在的View不会被移除。

这四种方法其实都是调用了PhoneWindow.java中的方法，通过源码我们可以发现setContentView(View view, ViewGroup.LayoutParams params)和setContentView(@LayoutRes int layoutResID)的步骤基本上是一样的，只不过是在添加到布局的时候，前者因为已经获得了View的实例，因此用的是addView的方法，而后者因为需要先inflate，所以，使用的是LayoutInflater。


