# 继承Thread类创建线程类

1. 定义Thread类的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run()方法称为执行体。

2. 创建Thread子类的实例，即创建了线程对象。

3. 调用线程对象的start()方法来启动该线程。

# 通过Runnable接口创建线程类

1. 定义runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体。

2. 创建Runnable实现类的实例，**并依此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。**

3. 调用线程对象的start()方法来启动该线程。

# 通过Callable和Future创建线程
1. 创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。

2. 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。

3. 使用FutureTask对象作为Thread对象的target创建并启动新线程。

4. 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值，调用get()方法会阻塞线程。

# 创建线程的三种方式的对比

**采用实现Runnable、Callable接口的方式创见多线程时，优势是：**

    线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。
    在这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想。

**劣势是：**

    编程稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法。

**使用继承Thread类的方式创建多线程时优势是：**
    编写简单，如果需要访问当前线程，则无需使用Thread.currentThread()方法，直接使用this即可获得当前线程。

**劣势是：**

    线程类已经继承了Thread类，所以不能再继承其他父类。
