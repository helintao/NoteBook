# AsyncTask 

AsyncTask 是一个抽象类。

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



