# Gazebo 世界（Worlds）

本主题提供 PX4 支持的 [Gazebo](../simulation/gazebo.md) 世界的有关图像/信息。

The [empty.world](#empty_world) is spawned by default, though this may be overridden by a [model specific world](#model_specific_worlds). Developers can also manually specify the world to load: [Gazebo Simulation > Loading a Specific World](../simulation/gazebo.md#set_world).

The source code for supported worlds can be found on GitHub here: [PX4/PX4-SITL_gazebo/tree/main/worlds](https://github.com/PX4/PX4-SITL_gazebo/tree/main/worlds).

<a id="empty_world"></a>

## Empty (Default)

[PX4/PX4-SITL_gazebo/tree/main/worlds/empty.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/empty.world)

![空白](../../assets/simulation/gazebo/worlds/empty.png)

## 海湾

[PX4/PX4-SITL_gazebo/tree/main/worlds/baylands.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/baylands.world)

![海湾世界](../../assets/simulation/gazebo/worlds/baylands.jpg)

## KSQL 机场

[PX4/PX4-SITL_gazebo/tree/main/worlds/ksql_airport.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/ksql_airport.world)

![KSQL机场世界](../../assets/simulation/gazebo/worlds/ksql_airport.jpg)

## 麦克米兰机场

[PX4/PX4-SITL_gazebo/tree/main/worlds/mcmillan_airfield.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/mcmillan_airfield.world)

![麦克米兰机场世界](../../assets/simulation/gazebo/worlds/mcmillan_airfield.jpg)

## Safe Landing

[PX4/PX4-SITL_gazebo/tree/main/worlds/safe_landing.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/safe_landing.world)

![Safe Landing World](../../assets/simulation/gazebo/worlds/safe_landing.png)

## 索诺玛赛道

[PX4/PX4-SITL_gazebo/tree/main/worlds/sonoma_raceway.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/sonoma_raceway.world) ![Sonoma_Raceway](../../assets/simulation/gazebo/worlds/sonoma_raceway.png)

## 仓库

[PX4/PX4-SITL_gazebo/tree/main/worlds/warehouse.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/warehouse.world)

![仓库](../../assets/simulation/gazebo/worlds/warehouse.png)

## 约塞米蒂

[PX4/PX4-SITL_gazebo/tree/main/worlds/yosemite.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/yosemite.world)

![约塞米蒂](../../assets/simulation/gazebo/worlds/yosemite.jpg)

<a id="model_specific_worlds"></a>

## Model Specific Worlds

Some [vehicle models](../simulation/gazebo_vehicles.md) rely on the physics / plugins of a specific world. The PX4 toolchain will automatically spawn a world that has the same name as the vehicle model if one exists (instead of the default **empty.world**):

The model specific worlds are:
- [boat.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/boat.world): Includes a surface to simulate buoyancy of the [boat](../simulation/gazebo_vehicles.md#usv).
- [uuv_hippocampus.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/uuv_hippocampus.world): An empty world used to simulate an underwater environment for the [HippoCampus UUV](../simulation/gazebo_vehicles.md#uuv).
- [typhoon_h480.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/typhoon_h480.world): Used by [Typhoon H480 (Hexrotor)](../simulation/gazebo_vehicles.md#typhoon_h480) vehicle model and includes a video widget to enable / disable video streaming. The world includes a gazebo plugin for a simulated camera.
- [iris_irlock.world](https://github.com/PX4/PX4-SITL_gazebo/blob/main/worlds/iris_irlock.world): Includes a IR beacon for testing [precision landing](../advanced_features/precland.md).
