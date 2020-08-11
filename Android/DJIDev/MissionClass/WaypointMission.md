# WaypoingMission
航点任务

## Members

- enum WaypointMissionExecuteState

    当前航点任务状态

    |参数|功能|
    |-|:-|
    |INITIALIZING|航点任务正在初始化，这意味着任务已开始并且飞机将进入第一个航点|
    |MOVING|飞机目前正朝着任务的下一个航点前进。将WaypointMissionFlightPathMode设置为NORMAL时会发生这种情况|
    |CURVE_MODE_MOVING|飞机目前正在移动。将WaypointMissionFlightPathMode设置为CURVED时会发生这种情况|
    |CURVE_MODE_TURNING|飞机目前正在转弯。将WaypointMissionFlightPathMode设置为CURVED时会发生这种情况|
    |BEGIN_ACTION|飞机已经到达航路点，已经旋转到新的航向，现在正在处理动作。该状态将在航点动作开始执行之前被调用，并将在每个航点动作中发生|
    |DOING_ACTION|飞机在航路点上并且正在执行动作|
    |FINISHED_ACTION|飞机在航路点上并已完成执行当前航路点的动作。每个航点动作都会发生一次此状态|
    |RETURN_TO_FIRST_WAYPOINT|飞机已返回第一个航点。将getFinishedAction设置为RETURN_TO_FIRST_WAYPOINT时会发生这种情况|
    |PAUSED|用户已中止当前任务|

- enum WaypointMissionFinishedAction

    任务完成后采取的行动

    |参数|功能|
    |-|:-|
    |NO_ACTION|任务完成将不会采取进一步的行动。此时，可以通过遥控器控制飞机|
    |GO_HOME|任务完成后，飞机将返航。如果飞机离返航点的距离超过20m，它将返航并着陆。否则，它将直接降落在当前位置|
    |AUTO_LAND|飞机将在最后一个航点自动降落|
    |GO_FIRST_WAYPOINT|飞机将返回其第一个航点并悬停就位|
    |CONTINUE_UNTIL_END|当飞机到达其最终航路点时，它将悬停而不会结束任务。操纵杆仍可用于将飞机沿其先前的航路点拉回。该任务可以结束的唯一方法是调用stopMission|

- enum WaypointMissionHeadingMode

    当前航路点任务航向模式

    |参数|功能|
    |-|:-|
    |AUTO|飞机的航向将始终在飞行方向上|
    |USING_INITIAL_DIRECTION|到达第一个航点时，飞机的航向将设置为航向。在到达第一个航路点之前，可以通过遥控器控制飞机的航向。当飞机到达第一个航点时，其航向将被固定。|
    |CONTROL_BY_REMOTE_CONTROLLER|飞机的航向将由遥控器控制|
    |USING_WAYPOINT_HEADING|在两个相邻航路点之间行驶时，飞机的航向将逐渐设置为下一个航路点|
    |TOWARD_POINT_OF_INTEREST|飞机的航向将始终朝着兴趣点前进|

-  enum WaypointMissionFlightPathMode

    航点任务飞行路径模式。

    |参数|功能|
    |-|:-|
    |
NORMAL|飞行路径将是正常的，并且飞机将以直线从一个航点移到下一个航点|
    |CURVED|飞行路径将是弯曲的，并且飞机将以弯曲的运动从一个航点移动到下一个航点，并遵循cornerRadiusInMeters设置在中Waypoint|

- enum WaypointMissionGotoWaypointMode

    该枚举将确定无人机执行飞行时的到航点模式。

    |参数|功能|
    |-|:-|
    |
SAFELY|安全地到达航路点。如果当前高度低于航路点的高度，则飞机将升至航路点的相同高度。然后，它从当前高度转到航路点坐标，然后前进到航路点的高度。|
    |POINT_TO_POINT	|从当前飞机点直接转到航点|

- DJIError checkParameters()

    在将任务配置载入之前，检查其配置是否有效

- int getWaypointCount()

    航路点任务中的航路点数

- int getMissionID()

    获取任务ID

- @Size(max = MAX_WAYPOINT_COUNT, min = MIN_WAYPOINT_COUNT)
 List\<Waypoint> getWaypointList()

    返回任务中所有航路点的集合

- float getMaxFlightSpeed()

    当飞机在航点之间行驶时，您可以使用遥控器上的油门操纵杆来抵消其速度。getMaxFlightSpeed是将操纵杆推到最大偏转位置时的偏移量。例如，如果maxFlightSpeed为10m/s，则将油门操纵杆完全向上推将使飞机速度增加10m/s，而向下推将使飞机速度减去10m/s。如果遥控器摇杆未处于最大偏转位置，则偏移速度将以1000步的分辨率插入\[0，getMaxFlightSpeed]之间。如果偏移速度为负，则飞机将向后飞行至先前的航路点。当到达第一个航路点时，它将悬停在适当的位置，直到施加正速度。getMaxFlightSpeed范围为\[2,15]m/s。

- float getAutoFlightSpeed()

    飞机在\[-15、15]m/s范围内的航点之间移动时的基本自动速度。飞机的实际速度是基本自动速度和遥控器上的油门操纵杆给出的速度控制的结合。如果getAutoFlightSpeed>0:实际速度为getAutoFlightSpeed+游戏杆速度（最大合计为getMaxFlightSpeed）如果getAutoFlightSpeed=0:实际速度仅由遥控器游戏杆控制。如果getAutoFlightSpeed<0，并且飞机在第一个航点，则飞机将悬停在适当位置，直到遥控器操纵杆将速度设为正。

- WaypointMissionGotoWaypointMode getGotoFirstWaypointMode()

    定义飞机如何从当前位置前往第一个航点。默认值为SAFELY。

- WaypointMissionFinishedAction getFinishedAction()

    航点任务完成后，飞机将采取的行动

- WaypointMissionHeadingMode getHeadingMode()

    飞机在航点之间移动时的航向。默认值为AUTO。

- LocationCoordinate2D getPointOfInterest()

    在航点任务期间，飞机的航向将固定在兴趣点位置。在getHeadingMode为TOWARD_POINT_OF_INTEREST时使用。

- WaypointMissionFlightPathMode getFlightPathMode()

    飞机在航路点之间遵循的路径。飞机可以直接在航路点之间直线飞行，也可以在弯道中的航路点附近转弯，航路点位置定义了弯路的一部分。

- boolean isExitMissionOnRCSignalLostEnabled()

    确定在飞机与遥控器之间的连接丢失时任务是否应该停止。默认值为false。

- boolean isGimbalPitchRotationEnabled()

    true如果在航路点任务期间可以控制万向节俯仰旋转。当时true，gimbalPitch在Waypoint用于控制云台俯仰。

- int getRepeatTimes()

    获取任务的重复次数