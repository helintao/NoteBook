# Builder
构建航点任务

## Method

- Builder addWaypoint(@NonNull Waypoint waypoint)

    添加一个航点到航点任务

- Builder insertWaypoint(@NonNull Waypoint waypoint, int index)

    插入一个航点到航点任务

- Builder waypointCount(int waypointCount)

    航点任务中的航点数，必须设置为添加到航点任务中的任务的个数

- int getWaypointCount()

    获取航点任务中的航点个数

- Builder waypointList(@NonNull @Size(max = MAX_WAYPOINT_COUNT, min = MIN_WAYPOINT_COUNT）List\<Waypoint> waypointList)

    将航点集合添加到任务中

- int getMissionID()

    获取任务ID

- Builder setMissionID(@IntRange(from = 0, to = 65535) int id)

    设置任务ID

- float calculateTotalDistance()

    计算航点列表的总距离

- float getLastCalculatedTotalDistance()

    获得最后计算出的航点列表的总距离

- Float calculateTotalTime()

    计算航点列表的总时间

- Float getLastCalculatedTotalTime()

    获取最后计算出的航点列表的总时间

- Builder maxFlightSpeed(@FloatRange(from = MIN_FLIGHT_SPEED, to = MAX_FLIGHT_SPEED)float maxFlightSpeed)

    设置最大飞行速度。
    当飞机在航点之间行驶时，您可以使用遥控器上的油门操纵杆来抵消其速度。getMaxFlightSpeed是将操纵杆推到最大偏转位置时的偏移量。例如，如果maxFlightSpeed为10m/s，则将油门操纵杆完全向上推将使飞机速度增加10m/s，而向下推将使飞机速度减去10m/s。如果遥控器摇杆未处于最大偏转位置，则偏移速度将以1000步的分辨率插入\[0，getMaxFlightSpeed]之间。如果偏移速度为负，则飞机将向后飞行至先前的航路点。当到达第一个航路点时，它将悬停在适当的位置，直到施加正速度。getMaxFlightSpeed范围为\[2,15]m/s。

- float getMaxFlightSpeed()

    获取最大飞行速度

- Builder autoFlightSpeed(@FloatRange(from = MIN_AUTO_FLIGHT_SPEED, to ==MAX_AUTO_FLIGHT_SPEED) float autoFlightSpeed)

    设置自动飞行速度

- float getAutoFlightSpeed()

    获取自动飞行速度

- Builder gotoFirstWaypointMode(@NonNull WaypointMissionGotoWaypointMode gotoFirstWaypointMode)

    定义飞机如何从当前位置飞往第一个航点，默认为SAFELY

- WaypointMissionGotoWaypointMode getGotoFirstWaypointMode()

    获取飞机从当前位置飞往第一个航点的模式

- Builder finishedAction(@NonNull WaypointMissionFinishedAction finishedAction)

    设置飞机完成任务后的动作

- WaypointMissionFinishedAction getFinishedAction()

    获取飞机完成任务后的动作

- Builder headingMode(@NonNull WaypointMissionHeadingMode headingMode)

    飞机在航点之间移动时的航向，默认为AUTO

- WaypointMissionHeadingMode getHeadingMode()

    获取飞机在航点之间移动时的航向

- Builder setPointOfInterest(@NonNull LocationCoordinate2D pointOfInterest)

    在航点任务期间，飞机的航向将固定在兴趣点位置。当getHeadingMode是 TOWARD_POINT_OF_INTEREST时调用

- LocationCoordinate2D getPointOfInterest()

    获取在航点任务期间，飞机的航向将固定在兴趣点位置

- Builder flightPathMode(@NonNull WaypointMissionFlightPathMode flightPathMode)

    飞机在航路点之间遵循的路径。飞机可以直接在航路点之间直线飞行，也可以在弯道中的航路点附近转弯，航路点位置定义了弯路的一部分。

- WaypointMissionFlightPathMode getFlightPathMode()

    获取飞机航点路径模式

- Builder setExitMissionOnRCSignalLostEnabled(boolean enabled)

    设置飞机与遥控器之间的连接丢失时任务是否应该停止，默认值为false

- boolean isExitMissionOnRCSignalLostEnabled()

    判断飞机与遥控器之间的连接丢失时任务是否应该停止

- Builder setGimbalPitchRotationEnabled(boolean gimbalPitchRotationEnabled)

    设置在航路点任务期间是否可以控制万向节俯仰旋转

- boolean isGimbalPitchRotationEnabled()

    判断在航路点任务期间是否可以控制万向节俯仰旋转

- Builder repeatTimes(@IntRange(from = MIN_REPEAT_TIME, to = MAX_REPEAT_TIME) int repeatTimes)

    设置任务重复执行的次数，0表示只执行一次

- int getRepeatTimes()

    获取任务重复执行的次数

- boolean isMissionComplete()

    判断任务信息是否完整

- WaypointMission build()

    构建WaypointMission对象