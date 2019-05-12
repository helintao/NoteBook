# AsyncTask 

AsyncTask 是一个抽象类。AsyncTask内部原理是Thread+Handler

三个参数(泛型参数)
```java
Params//在执行AsyncTask时需要传入的参数，可用于后台任中使用
Progress//后台任务执行时，如果需要在界面上显示当前进度，则使用这里指定的泛型作为进度单位
Result//当任务执行完毕后，如果需要对结果进行返回，则使用这里指定的泛型作为返回值类型。
```

一个简单的自定义AsyncTask
```java
class Download extends AsyncTask<Void,Integer,Boolean>{
    ......
}
参数一：Void，表示在执行AsyncTask的时候不需要传入参数给后台任务。
参数二：Integer，表示使用整型数据来作为进度显示单位。
参数三：Boolean，表示使用布尔值数据来反馈执行结果。
```

需要重写的四个方法

- onPreExecute();  
这个方法会在后台任务**开始执行之前调用**，用于进行一些界面上的初始化操作，比如显示一个进度对话框等。*此方法运行在UI线程*。

- doInBackground(Params...);  
这个方法运行在*子线程*。此方法**执行耗时操作**，任务一旦完成就可以通过return语句来将任务的执行结果进行返回。在这个方法中是不可以进行UI操作的，如果需要更新UI元素，比如说反馈当前任务的执行进度，可以调用publishProgress(Progress...)方法来完成。

- onProgressUpdate(Params...);
当在后台任务中**调用了publishProgress(Progress...)方法后**，这个方法就很快会被调用，方法中携带的参数就是在后台任务中传递过来的。在这个方法中可以对UI进行操作，利用参数中的数值就可以对界面元素进行相应的更新。*此方法运行在UI线程*。

- onPostExecute(Result);
当后台任务执行完毕并通过**return语句进行返回时**，这个方法就很快会被调用。返回的数据会作为参数传递到此方法中，可以利用返回的数据来进行一些UI操作，比如说提醒任务执行的结果，以及关闭掉进度条对话框等。*此方法运行在UI线程*。

执行任务时只需调用execute()方法。

# AsyncTask源码解析  

- execute()方法
  ```java
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);//sDefaultExecutor串行的线程池
    }

    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;//设置状态为运行态

        onPreExecute();

        mWorker.mParams = params;
        exec.execute(mFuture);//mFuture是一个异步处理获取执行结果类的实例

        return this;
    }
  
  ```
  分析：可以看出，调用execute()方法后，内部调用了executeOnExecutor(sDefaultExecutor, params)方法。在executeOnExecutor方法中，首先判断AsyncTask是否第一次执行，否则抛出异常。如果第一次运行，则把状态设为运行态。接着调用onPreExecute()方法(我们上面总结的第一个方法在这个方法里调用啦)。最后线程池开始执行，即exec.execute(mFuture)开始执行，*exec是一个串行的线程池，一个进程中所有的AsyncTask全部在这个串行的线程池中排队执行*

- 线程池的分析
  
  ```java
    public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

    private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;//volatile修饰符表示防止编译器对代码进行优化。

    private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();//创建任务队列

        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            //把FutureTask对象插入到任务队列中。
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();//当此任务执行完后，继续执行下一个任务。
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();//当mActive为空，执行下一个任务
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);//执行任务
            }
        }
    }
  ```

  分析：
  系统把AsyncTask的Params参数封装为FutureTask对象，接着把这个FutureTask会交给SerialExecutor  的execute方法去处理。  
  在execute方法中：首先会把FutureTask对象插入到任务队列mTasks中。如果这个时候没有正在执行的AsyncTask任务，就会调用scheduleNext()方法来执行下一个AsyncTask任务。当一个AsyncTask任务执行完成后，AsyncTask会继续执行其他任务。

- AsyncTask的两个线程（SerialExecutor和THREAD_POOL_EXECUTOR
  
  SerialExecutor用于任务的排列。THREAD_POOL_EXECUTOR用于真正的执行任务。
  ```java
    
    public static final Executor THREAD_POOL_EXECUTOR;
    //创建线程池
    static {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                sPoolWorkQueue, sThreadFactory);
        threadPoolExecutor.allowCoreThreadTimeOut(true);
        THREAD_POOL_EXECUTOR = threadPoolExecutor;
    }

  ```

- AsyncTask的构造函数
```java

    /**
     * Creates a new asynchronous task. This constructor must be invoked on the UI thread.
     */
    public AsyncTask() {
        this((Looper) null);
    }

    /**
     * Creates a new asynchronous task. This constructor must be invoked on the UI thread.
     *
     * @hide
     */
    public AsyncTask(@Nullable Handler handler) {
        this(handler != null ? handler.getLooper() : null);
    }

    /**
     * Creates a new asynchronous task. This constructor must be invoked on the UI thread.
     *
     * @hide
     */
    public AsyncTask(@Nullable Looper callbackLooper) {

        mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
            ? getMainHandler()
            : new Handler(callbackLooper);//如果callBackLooper为空或者等于主线程的Looper,则持有主线程的Handler引用，否则持有一个新的对应线程的Handler（传入callbackLooper参数）。

        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);//表示当前任务已经调用过了
                Result result = null;
                try {
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);//设置进程优先级
                    //noinspection unchecked
                    result = doInBackground(mParams);
                    Binder.flushPendingCommands();
                } catch (Throwable tr) {
                    mCancelled.set(true);// 若运行异常,设置取消的标志
                    throw tr;
                } finally {
                    postResult(result);//返回结果
                }
                return result;
            }
        };

        mFuture = new FutureTask<Result>(mWorker) {

            // done（）FutureTask内的Callable执行完后的调用方法，即mWorker的run方法.
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }

    //WorkerRunnable抽象类继承Callable接口。
    private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
        Params[] mParams;
    }

    private void postResultIfNotInvoked(Result result) {
        final boolean wasTaskInvoked = mTaskInvoked.get();
        //若任务无被执行,将未被调用的任务的结果通过Handler传递到对应线程
        if (!wasTaskInvoked) {
            postResult(result);
        }
    }

    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
```

分析：
可以看出，创建AsyncTask必须在UI线程上创建。  
在AsyncTask构造方法中两段实例化代码。  
  - 首先来分析下new WorkerRunnable<Params, Result>() {......}  
    将mTaskInvoked设置为true。表示当前任务已经调用过了。接着执行AsyncTask的doInBackground()方法（doInBackground()是在此被调用的。）最后将返回值传递给postResult方法。postResult方法如上所示，postResult方法会通过mHandler发送一个MESSAGE_POST_RESULT的消息。这样就通过mHandler切换到对应线程上去处理返回的结果。

  - FutureTask的run方法会调用mWorker的call方法，因此mWorker的call方法最终会在线程中执行。
  ```java
  //FutureTask.class

    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }

    public void run() {
        if (state != NEW ||
            !U.compareAndSwapObject(this, RUNNER, null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }

    protected void set(V v) {
        if (U.compareAndSwapInt(this, STATE, NEW, COMPLETING)) {
            outcome = v;
            U.putOrderedInt(this, STATE, NORMAL); // final state
            finishCompletion();
        }
    }

    private void finishCompletion() {
        // assert state > COMPLETING;
        for (WaitNode q; (q = waiters) != null;) {
            if (U.compareAndSwapObject(this, WAITERS, q, null)) {
                for (;;) {
                    Thread t = q.thread;
                    if (t != null) {
                        q.thread = null;
                        LockSupport.unpark(t);
                    }
                    WaitNode next = q.next;
                    if (next == null)
                        break;
                    q.next = null; // unlink to help gc
                    q = next;
                }
                break;
            }
        }

        done();

        callable = null;        // to reduce footprint
    }
  ```
  分析从FutureTask代码中就可发现，FutureTask的run方法会调用mWorker的call方法，在mWorker的call方法执行完毕后，会调用set(result)方法。set(result)中调用了finishCompletion()，最后finishCompletion()中done方法。done内部就是返回结果之类的操作了。

- onProgressUpdate(Params...)

  在doInBackground()方法调用 publishProgress(params)方法
  ```java
  protected final void publishProgress(Progress... values) {
      if (!isCancelled()) {
          getHandler().obtainMessage(MESSAGE_POST_PROGRESS,
                  new AsyncTaskResult<Progress>(this, values)).sendToTarget();
      }
  }
  
  private static class InternalHandler extends Handler {
      public InternalHandler(Looper looper) {
          super(looper);
      }
  
      @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
      @Override
      public void handleMessage(Message msg) {
          AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
          switch (msg.what) {
              case MESSAGE_POST_RESULT:
                  // There is only one result
                  result.mTask.finish(result.mData[0]);
                  break;
              case MESSAGE_POST_PROGRESS:
                  result.mTask.onProgressUpdate(result.mData);//调用onProgressUpdate方法。
                  break;
          }
      }
  }
  ```
  分析：通过上面的源码可以发现，当我们调用了publishProgress(params)方法时，内部会通过Handler机制发送一个MESSAGE_POST_PROGRESS消息，然后就切换到了主线程执行onProgressUpdate(Params...)方法。

  - onPostExecute(Result)
  ```java
    private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }
  ```
  分析：综合上面的代码分析，当任务完成时，内部会通过Handler机制发送一个MESSAGE_POST_RESULT消息，然后就切换到了主线程执行调用finish(result.mData[0])方法，在finish(result)中执行onPostExecute(Result)方法。并设置状态为完成态。



总结： 
  - 实例化AsyncTask，在实例化的时候，获得到了Handler，创建了FutureTask对象和WorkerRunnable对象，其中在WorkerRunnable对象中的实现的call方法里，调用了doInBackground()方法，并在doInBackground()方法完成后通过postResult()方法将结果通过Handler发送给了主线程。在FutureTask对象中传入WorkerRunnable对象，最终在内部调用WorkerRunnable对象中的call方法。
  - 当我们调用execute方法的时候，内部会调用executeOnExecutor方法，并传入Params。在executeOnExecutor中会对AsyncTask的状态进行判断，如果正在运行或者结束则抛出异常。反之，改变状态为运行态，调用onPreExecute方法，将Params传入到mWorker中，最后SerialExecutor的实例调用execute(mFuture)方法，并传入mFuture。  
  - 在execute(final Runnable r)中,将传进来的mFuture插入到任务队列中，如果这个时候没有正在执行的AsyncTask任务，就会调用scheduleNext()方法来执行下一个AsyncTask任务。当一个AsyncTask任务执行完成后，AsyncTask会继续执行其他任务。  
  执行任务的语句：THREAD_POOL_EXECUTOR.execute(mActive);将任务交给线程池去处理。处理结果通过Handler机制发送给主线程。这就是整个流程。  



