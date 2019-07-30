# AndroidManifest.xml中指定launchMode
## standard
标准模式，每次启动Activity都会创建一个新的Activity实例，并且将其压入任务栈栈顶，而不管这个 Activity 是否已经存在，都会执行onCreate() ->onStart() -> onResume

## singleTop
栈顶复用模式，如果新Activity已经位于栈顶，那么此Activity不会被重新创建，同时Activity的 onNewIntent方法会被回调，如果Activity已经存在但是不再栈顶，那么和standard模式一样。
如果Activity当前是onResume状态，那么调用后会执行onPause() -> onNewIntent() -> onResume()。

## singleTask
栈内复用模式，创建这样的Activity，系统会确认它所需任务栈是否已经创建，否则先创建任务栈，然后放入Activity，如果栈中已经有一个Activity实例，那么会做两件事：
- 这个Activity会回到栈顶执行onNewIntent
- 清理在当前Activity上面的所有Activity

*如果栈中已经有一个Activity实例，这个判断条件的标准是由android:taskAffinity决定的*

## singInstance
这种模式的Activity只能单独位于一个任务栈内，由于栈内的复用特性，后续请求均不会创建新的Activity，除非这个独特的任务栈被系统销毁了。

# 在Intent当中指定启动模式
## FLAG_ACTIVITY_NEW_TASK
和singleTask行为相同，这里需要注意 affinity 的声明。

## FLAG_ACTIVITY_SINGLE_TOP
和singleTop行为相同。

## FLAG_ACTIVITY_CLEAR_TASK
和 FLAG_ACTIVITY_NEW_TASK 合用，这个Activity会新起一个栈，原来栈被清空，栈中的Activity也被销毁。

## FLAG_ACTIVITY_CLEAR_TOP
会清除这个Activity之上所有的Activity

## FLAG_ACTIVITY_REORDER_TO_FRONT
上面的FLAG_ACTIVITY_CLEAR_TOP是把位于目标Activity之上的Activity都销毁，而则个FLAG则是对栈重新排序，把目标Activity移到最前台，其它的位置不变。

例如：  
栈内activity：MainActivity、FiratActivity、SecondActivity。  
FirstActivity启动模式为FLAG_ACTIVITY_REORDER_TO_FRONT。则FirstActivity启动后，栈内activity：MainActivity、SecondActivity、FiratActivity。

# AndroidManifest中的属性
## alwaysRetainTaskState
这个标志只对根Activity有用，默认情况下，当我们的应用在后台一段时间，它会销毁该Task除了根以外的所有Activity，如果我们希望保持这个Task的原有状态，那么给这个Task的根Activity设置这个属性，默认值是false。

## clearTaskOnLaunch
从桌面启动该Activity的时候会清空该Task除了根Activity外的所有Activity，我们从Main -> Second -> Third，此时栈内有3个Activity，按Home回到桌面后，点图标重新进入，此时Task只剩下根Activity 了。

## finishOnTaskLaunch
这个和上面类似，但是它对根Activity无效，我们给SecondActivity设置这个属性，先启动到ThirdActivity。接着，我们按Home回到桌面，点图标重新进入，可以看到SecondActivity没有了。

## noHistory
Activity在不可见之后，不保存记录。
- 第一种情况，我们给SecondActivity设置这个属性，接着从Main -> Second -> Third，然后按Back返回，此时的生命周期为：

```
ThirdActivity#onPause
MainActivity#onRestart
MainActivity#onStart
MainActivity#onResume
SecondActivity#onDestroy
ThirdActivity#onStop
ThirdActivity#onDestroy
```

- 第二种情况，如果我们在ThirdActivity时，不是按Back，而是按Home到桌面，会调用：
```
ThirdActivity#onPause
SecondActivity#onDestroy
ThirdActivity#onStop
```

- 第三种情况，我们给根MainAcitivity设置这个属性，启动它后退出：
```
MainActivity#onCreate
MainActivity#onStart
MainActivity#onResume
MainActivity#onDestory
```