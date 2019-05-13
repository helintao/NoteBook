# HandlerThread

HandlerThread继承了Thread,它是一种可以使用Handler的Thread。  
HandlerThread实现其实就在run方法中通过调用Looper.prepare()来创建消息队列，并通过Looper.loop()来开启消息循环。
```java
public void run() {
    mTid = Process.myTid();
    Looper.prepare();
    synchronized (this) {
        mLooper = Looper.myLooper();
        notifyAll();
    }
    Process.setThreadPriority(mPriority);
    onLooperPrepared();
    Looper.loop();
    mTid = -1;
}
```

分析：与普通的Thread不同的是，普通的Thread主要用于在run方法中执行一个耗时任务。而HandlerThread执行了一个具体的任务。

HandlerThread的run方法是一个无限循环，在明确不需要再使用HandlerThread时，可以通过它的quit或者quitSafely方法来终止线程的执行。

```java
    /**
     * Call back method that can be explicitly overridden if needed to execute some
     * setup before Looper loops.
     */
    protected void onLooperPrepared() {
    }
```
分析：如果要在执行Looper.loop()之前进行某些操作，则可以通过重写此回调方法达到目的。