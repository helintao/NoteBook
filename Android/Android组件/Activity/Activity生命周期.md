# Activity生命周期
![activity生命周期](../../img/activity_life.png)

# protected void onCreate(Bundle savedInstanceState)
- 该方法被回调时，一个意味着新的Activity被创建了。
- 当前由于处于一个新的Activity实体对象当中，所有的东西都是未初始化的，一般我们需要做的事情包括调用setContentView方法设置该Activity的布局，初始化类成员变量。
- onCreate(xxx)方法执行完之后，Activity就展示进入了Created状态，然而它并不会在这个状态停留，会系统-接着回调onStart()方法由Created状态展示进入到Started状态。
- 注意到，onCreate(xxxx)是所有这些回调方法中唯一有参的，参数该保存了上次由于Activity被动回收时所保存的数据。

# protected void onStart()
- onStart()方法执行完后，Activity就展示进入了Started状态，它也同样不会在该状态停留，接着而是回调onResume()方法展示进入Resumed状态。
- onStart()被回调的情况有两种：
    - 从Created状态过来
    - 从Stopped状态过来，这种从状态过来还会先经过onRestart()方法。
    它由于从也会Stopped状态跳转过来，因此如果在我们onStop()当中反注册了一些广播，或者释放了一些资源，那么在这里需要重新注册或者初始化。
- onStart()到onResume()的过程中，还可能会回调onRestoreInstanceState/onPostCreate/onNewIntent/onActvitiyResult这几个方法。
- 如果应用退到后台，再次被启动（onNewIntent），或者通过startActivityForResult方法启动另一个 Activity得到结果返回（onActivityResult）的时候，在onStart()方法当中得到的并不是这两个回调的最新结果，因为上面的这两个方法并没有调用，而在onResume()当中，这点是可以得到保证的。

# protected void onResume()
- 该方法被回调时，意味着Activity已经完全可见了，但这个完全可见的概念并不等同于Activity所属的Window被Attach了，因为在Activity初次启动的时候，Attach的操作是在回调onResume()之后的。
- 在onResume()方法被回调时，由于DecorView并不一定Attach了，因此这时候我们获取布局内某些View 的宽高得到的值有可能是不正确的，既然onResume()当中不能保证，那么onStart()方法也是同理，所有需要在attachToWindow之后才能执行或是期望得到正确结果的操作都需要注意这一点。
- 在onResume当中，我们一般会做这么一些事：在可见时重新拉取一些需要及时刷新的数据、注册 ContentProvider的监听，最重要的是在onPause()中的一些释放操作要在这里面恢复回来。

# protected void onPause()
- 该方法被回调时，意味着Activity部分不可见，例如一个半透明的界面覆盖在了上面，这时候只要Activity仍然有一部分可见，那么它会一直保持在Paused状态。
- 如果用户从Paused状态回到Resumed状态，只会回调onResume方法。
- 在onPause()方法中，应该暂停正在进行的页面操作，例如正在播放的视频，或者释放相机这些多媒体资源。
- 在onPause()当中，可以保存一些必要数据到持久化的存储，例如正在编写的信息草稿。
- 不应该在这里执行耗时的操作，因为新界面启动的时候会先回调当前页面的onPause()方法，所以如果进行了耗时的操作，那么会影响到新界面的启动时间，官方文档的建议是这些操作应该放到 onStop()当中去做，其实在onStop()中也不应当做耗时的操作，因为它也是在主线程当中的，而在主线程中永远不应该进行耗时的操作。
- 释放系统资源，例如先前注册的广播、使用的传感器（如GPS）、以及一些仅当页面获得焦点时才需要的资源。
- 当处于Paused状态时，Activity的实例是被保存在内存中的，因此在其重新回到Resumed状态的过程中，不需要重新初始化任何的组件。

# protected void onStop()
- 该方法被回调时，表明Activity已经完全不可见了。
- 在任何场景下，系统都会先回调onPause()，之后再回调onStop()。
- 官方文档有谈到，当onStop()执行完之后，系统有可能会销毁这个Activity实例，在某些极端情况下，有可能不会回调onDestroy()方法，因此，我们需要在onStop()当中释放掉一些资源来避免内存泄漏，而 onDestory()中需要释放的是和Activity相关的资源，如线程之类的（这点平时在工作中很少用，一般我们都是在onDestroy()中释放所有资源，而且也没有去实现onStart()方法，究竟什么时候不会走onDestroy()，这点值得研究，目前的猜测是该Activity在别的地方被引用了，导致其不能回收）。
- 当Activity从Stopped状态回到前台时，会先回调onRestart()方法，然而我们更多的是使用onStart() 方法作为onStop()的对应关系，因为无论是从Stopped状态，还是从Created状态跳转到Resumed状态，都是需要初始化必要的资源的，而它们经过的共同路径是onStart()。

# protected void onDestroy()
- 该方法被回调时，表明Activity是真正要被销毁了，
- 此时应当释放掉所有用到的资源，避免内存泄漏。

