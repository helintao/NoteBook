# Window和WindowManager  

Window是一个抽象类，它的具体实现是PhoneWindow。  

WindowManager是外界访问Window的入口，Window可以通过WindowManager来创建。  

Window的具体实现位于WindowManagerService中，WindowManager和WindowManagerService的交互是一个IPC过程。  

Android的所有视图都是通过Window来呈现，Window实际是View的直接管理者。

WindowManager的真正实现类是WindowManagerImpl

Window的本质是其实就是一块显示区域，在Android中就是绘制的画布：Surface，当一块 Surface 显示在屏幕上时，就是用户所看到的窗口了。
WindowManagerService 添加一个窗口的过程，其实就是 WindowManagerService 为其分配一块 Surface 的过程。

一块块的Surface在WindowManagerService的管理下有序的排列在屏幕上，Android才得以呈现出多姿多彩的界面。

## 使用WindowManager添加一个Window
```java
    /**
    *此代码可以将一个Button添加到屏幕坐标为(100,300)的位置上。
    */
    mFloatingButton = new Button(this);
    mFloatingButton.setText("button");
    mLayoutParams = new WindowManager.LayoutParams(
        LayoutParams.WRAP_CONTENT,LayoutParams.WRAP_CONTENT,0,0,PixelFormat.TRANSPARENT);
    mLayoutParams.flags = LayoutParams.FLAG_NOT_TOUCH_MODAL
        | LayoutParams.FLAG_NOT_FOCUSABLE
        | LayoutParams.FLAG_SHOW_WHEN_LOCKED;
    mLayoutParams.gravity = Gravity.LEFT | Gravity.TOP;
    mLayoutParams.x = 100;
    mLayoutParams.y = 300;
    mWindowManger.addView(mFloatingButton,mLayoutParams);
```
Flags参数表示Window的属性，它有很多选项，通过这些选项可以控制Window的显示特征：

- FLAG_NOT_FOCUSABLE

  表示Window不需要获取焦点，也不需要接收各种输入事件，此标记会同时启用FLAG_NOT_TOUCH_MODAL,最终事件会直接传递给下层的具有焦点的Window。

- FLAG_NOT_TOUCH_MODAL

  系统会将当前Window区域外的单击事件传递给底层的Window，当前Window区域以内的单击事件则自己处理。这个标记很重要，一般来说都需要开启此标记，否则其他Window将无法收到单击事件。

- FLAG_SHOW_WHEN_LOCKED

  开启此模式可以让Window显示在锁屏的界面上。

Type参数表示Window的类型：

  - 应用Window：对应着一个Acivity。

  - 子Window:子Window不能单独存在，它需要附属在特定的父Window之中，比如常见的一些Dialog就是一个子Window。

  - 系统Window:需要声明权限在能创建的Window，比如Toast和系统状态栏这些都是系统Window。

## WindowManager所提供的常用的三个方法
```java
    public interface ViewManager{
        public void addView(View view,ViewGroup.LayoutParams params);
        public void updateViewLayout(View view,ViewGroup.LayoutParams params);
        public void removeView(View view);
    }
```