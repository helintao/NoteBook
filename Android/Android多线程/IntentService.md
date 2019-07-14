# IntentService

IntentService是一种特殊的Service。它继承了Service，是一个抽象类。  
IntentService可用于执行后台耗时的任务，当任务执行后它会自动停止。  
IntentService由于是服务的原因，它的优先级比单纯的线程要高很多。

# IntentService源码分析

```java
    @Override
    public void onCreate() {
        // TODO: It would be nice to have an option to hold a partial wakelock
        // during processing, and to have a static startService(Context, Intent)
        // method that would launch the service & hand off a wakelock.

        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }
```
分析：当IntentService被第一次启动时，它的onCreate方法会被调用，onCreate方法会创建一个HandlerThread，然后使用它的Looper来构造一个Handler对象mServiceHandler，这样通过mServiceHandler发送的消息最终都会在HandlerThread中执行。


每次启动IntentService，它的onStartCommand方法就会调用一次，IntentService在onStartCommand中处理每个后台任务的Intent。onStartCommand调用了onStart。
```java
    @Override
    public void onStart(@Nullable Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }
```
分析：IntentService仅仅通过mServiceHandler发送一个消息，这个消息会在HanderThread中被处理。mServiceHandler收到信息后，会将Intent对象传递给onHandlerIntent方法去处理。这个Intent对象的内容和外界启动IntentService(intent)中的intent的内容是完全一致的。通过Intent对象即可解析出外界启动IntentService时所传入的参数，通过这些参数就可以区分具体的后台任务，这样在onHandleIntent方法中就可以对不同的后台任务做处理。

```java
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }

    protected abstract void onHandleIntent(@Nullable Intent intent);

```
分析：当onHandlerIntent方法执行结束后，IntentService会通过stopSelf(int startId)方法来尝试停止服务。(stopSelf(int startId)会等待所有消息都处理完毕后才终止服务，而stopSelf()会立刻停止服务)。onHandleIntent方法是个抽象方法。它需要我们在子类中实现，*它的作用是从Intent参数中区分具体的任务并执行这些任务*。如果目前只有一个后台任务，那么onHandlerIntent方法执行完这个任务后，stopSelf(int startId)就会直接停止服务；如果目前存在多个后台任务，那么当onHandlerIntent方法执行完最后一个任务时，stopSelf(int startId)才会直接停止服务。

由于每执行一个后台任务就必须启动一次IntentService，而IntentService内部则通过消息的方式向HandlerThread请求执行任务，Handlr中的Looper是顺序处理消息的，这就意味着IntentService也是顺序执行后台任务的，当有多个后台任务同时存在时，这些后台任务会**按照外界发起的顺序排队执行**。

