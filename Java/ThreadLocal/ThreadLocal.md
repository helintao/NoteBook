## ThreadLocal  
ThreadLocal类似于是线程中存储数据的局部变量。

## ThreadLocal类  

- ThreadLocal类提供的几个方法
  ```Java
  public T get(){}//用来获取ThreadLocal在当前线程中保存的变量副本
  public void set(T value){}//用来设置当前线程中变量的副本
  public void remove(){}//用来移除当前线程中变量的副本
  protected T initialValue(){}//一般用来在使用时进行重写，它是一个延迟加载的方法。
  ```
  get方法的源码分析  
  ```java
    /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();//获取当前线程
        ThreadLocalMap map = getMap(t);//获取一个map，类型为ThreadLocalMap
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

    /**
     * Get the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param  t the current thread
     * @return the map
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;//返回当前线程中的一个成员变量ThreadLocalMap
    }

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. 
     *关于这个线程的ThreadLocal的值，这个map由ThreadLocal class维护*/
    ThreadLocal.ThreadLocalMap threadLocals = null;

    static class ThreadLocalMap {

        /**
         * The entries in this hash map extend WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e. entry.get()
         * == null) mean that the key is no longer referenced, so the
         * entry can be expunged from table.  Such entries are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }//继承WeakReference（弱引用），并且使用ThreadLocal作为键值
        ·
        ·
        ·
    ｝

    /**
     * Variant of set() to establish initialValue. Used instead
     * of set() in case user has overridden the set() method.
     *
     * @return the initial value
     */
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)//如果map不为空，就设置键值对，为空，再创建Map
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }

     /**
     * Create the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the map
     */
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
  ```
  总结：
    
    1. 每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前变量，value为变量副本（即T类型的变量）。  
    2. 初始时，在Thread里面，threadLocals为空，当通过ThreadLoacl变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。  
    
      ThreadLocal的get方法实际获得对应Thread中的ThreadLocalMap对象map（不存在则创建一个）,通过map获得table[] (类型   Entry[]), table[ThreadLocal对象的引用通过计算]得到对应的Entry。从而获取到value。（键值ThreadLocal对象，值Value）
