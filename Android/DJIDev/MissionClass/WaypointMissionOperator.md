# WaypointMissionOperator
航点任务操作员

## Member

- DJIError loadMission(@NonNull WaypointMission mission)

    加载任务（将任务从平板设备加载到遥控器设备内存中）。mission在WaypointMission执行完成后，该对象仍会保留在设备内存中。

- WaypointMission getLoadedMission()

    获取操作员当前加载的任务对象。当执行的加载任务被停止、完成或者中断，则getLoadedMission将会被重置为null.

- void getPreviousInterruption(CommonCallbacks.CompletionCallbackWith\<WaypointMissionInterruption> callback)

    获取上一个航点任务的中断。如果任务在完成前被中断，则飞机将会记录该中断信息。

- void uploadMission(@Nullable final CompletionCallback callback)

    开始上传getLoadedMission获取到的任务到飞机上。只有当getLoadedMission获取任务完成和getCurrentStates是READY_TO_UPLOAD时，才能上传。如果在上一个上载期间发生超时错误，则上载操作将从上一个断点恢复。成功上传任务后，WaypointMissionState将变为READY_TO_EXECUTE。

- void retryUploadMission(@Nullable CompletionCallback callback)

    重试上传任务。

- void downloadMission(@Nullable CompletionCallback callback)

    从飞机上下载每个航路点的信息，并将其保存到中getLoadedMission。如果开始下载操作，操作员将以getLoadedMission一个接一个升序的方式下载丢失的航点信息。如果getLoadedMission已经完成（包含所有航路点），则此方法将completion立即调用而不会出错。仅当getCurrentState以下之一时才可以调用它：EXECUTING--EXECUTION_PAUSED

- void startMission(@Nullable CompletionCallback callback)

    执行任务。只能在getCurrentState为READY_TO_EXECUTE时调用。成功执行任务后，getCurrentState将变为EXECUTING。

- void resumeMission(@Nullable CompletionCallback callback)

    恢复已暂停的任务。只能在getCurrentState为EXECUTION_PAUSED时调用。成功完成任务后，getCurrentState将变为EXECUTING。

- void pauseMission(@Nullable CompletionCallback callback)

    暂停执行任务。只能在WaypointMissionState为EXECUTING时调用。成功暂停任务后，getCurrentState将变为EXECUTION_PAUSED。

- void stopMission(@Nullable CompletionCallback callback)

    停止执行或暂停的任务。它只能在当getCurrentState是以下情况之一调用：EXECUTING、EXECUTION_PAUSED，停止成功后getCurrentState将成为READY_TO_UPLOAD。

- void setAutoFlightSpeed(@FloatRange(from = -15.0f, to = 15.0f) float speed,@Nullable CompletionCallback callback)

    设置自动飞行速度，范围\[-15.0f,15.0f]单位m/s。

- void getAutoFlightSpeed(CommonCallbacks.CompletionCallbackWith\<Float> callback)

    在执行任务是获取飞行速度。

- WaypointMissionState getCurrentState()

    获取当前操作员状态

- void addListener(@NonNull final WaypointMissionOperatorListener listener)

    添加监听

- void removeListener(@NonNull WaypointMissionOperatorListener listener)

    移除监听

