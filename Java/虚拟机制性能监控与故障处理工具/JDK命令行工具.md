# jps:虚拟机进程状况工具（JVM Process Status Tool）

列出正在运行的虚拟机进程，并显示虚拟机执行主类(Main Class,main()函数所在的类)名称以及这些进程的本地虚拟机唯一ID(Local Virtual Machine Identifier,LVMID)

## 命令格式：

jps [options] [hostid]

## jps工具的选项

|选项|作用|
|-|-|
|-q|只输出LVMID，省略主类的名称|
|-m|输出虚拟机进程启动时传递给主类main()函数的参数|
|-l|输出主类的全名，如果进程执行的是jar包，输出jar路径|
|-v|输出虚拟机进程启动时JVM参数|

# jstat:虚拟机统计信息监测工具(JVM Statistics Monitoring Tool)

用于监视虚拟机各种运行状态信息的命令行工具，它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行参数。

## 命令格式

jstat [option vmid [interval [s|ms] [count]]]

## jstat工具的选项

|选项|作用|
|-|-|
|-class|监视类装载、卸载数量、总空间以及类装载所耗费的时间|
|-gc|监视Java对状况、包括Eden区、两个survivor区、老年代、永久代等容量、已用空间、GC时间合计等信息|
|-gccapacity|监视内容与-gc基本相同，但输出主要关注java堆各个区域使用到的最大、最小空间|
|-gcutil|监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比|
|-gccause|与-gcutil功能一样，但是会额外输出导致上一次gc产生的原因|
|-gcnew|监视新生代GC状况|
|-gcnewcapacity|监视内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间|
|-gcold|监视老年代GC状况|
|-gcoldcapacity|监视内容与-gcold基本相同，输出主要关注使用到的最大、最小空间|
|-gcpermcapacity|输出永久代使用到的最大、最小空间|
|-compiler|输出JIT编译器编译过的方法、耗时等信息|
|-printcompilation|输出已经被JIT编译的方法|

# jinfo:Java配置信息工具（Configuration Info for Java）

实时地查看和调整虚拟机各项参数

## jinfo命令格式：

jinfo [option] pid

# jmap:Java内存映像工具（Memory Map for Java）

用于生成堆转储快照（一般称为heapdump或dump文件）。jmap的作用不仅仅是为了获取dump文件，它还可以查询finalize执行队列、Java堆和永久代的详细信息，如空间使用率，当前用的是哪种收集器等。

## jmap命令格式

jmap[option] vmid

## jmap工具主要选项

|选项|作用|
|-|-|
|-dump|生成Java堆转储快照。格式为：-dump:[live,]format=b,file=\<filsname>,其中live子参数说明是否只dump中存活对象|
|-finalizerinfo|显示在F-Queue中等待Finalizer线程执行finalize方法的对象，只在Linux/Solaris平台下有效|
|-heap|显示Java堆详细消息，如使用哪种回收器、参数配置、分代状况等。只在Linux/Solaris平台下有效。|
|||
|||
|||