**当系统内存不足时，Android系统将根据进程的优先级选择杀死一些不太重要的进程，优先级低的先杀死。进程优先级从高到低如下：**

# 前台进程
- 处于正在与用户交互的activity

- 与前台activity绑定的service

- 调用了startForeground()方法的service

- 正在执行onCreate(),onStart(),onDestory()的service。

- 进程中包括正在执行onReceive()方法的BroadcastReceiver。

# 可视进程
- 为处于前台，但仍然可见的activity（例如：调用了onPause()而没有调用onStop()的activity）。典型情况：运行activity时，弹出对话框(dialog等)，此时的activity虽然不是前台activity，但是仍然可见。

- 可见activity 绑定的Service。（处于上述情况下的activity所绑定的service）

*可视进程一般也不会被系统杀死，除非为了保证前台进程的运行不得已而为之。*

# 服务进程
- 已经启动的service

# 后台进程
- 不可见的activity。（调用了onStop()方法之后的activity）

*后台进程不会影响用户的体验，为了保证前台进程，可视进程，服务进程的运行，系统随时有可能杀死一个后台进程。当一个正确实现了生命周期的activity处于后台被杀死时，如果用户重新启动，会恢复之前的运行状态。*

# 空进程
- 任何没有活动的进程

*系统会杀死空进程，但这不会造成影响。空进程的存在无非为了一些缓存，以便于下次可以更快的启动。*