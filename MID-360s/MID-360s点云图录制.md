## 1. 使用模式说明

当前使用的硬件配置如下：

- `MID-360s` 雷达接开发板 `eth0`
    
- 开发板另一网口接 `R9000P` 笔记本
    
- 开发板负责运行 `livox_ros_driver2` 和 `FAST_LIO`
    
- 上位机 `WSL2` 负责运行 `rviz2`
    
- 录制文件保存在开发板本地
    
- 回放时由开发板离线重放 bag，`WSL2` 侧负责显示地图、位姿和轨迹


这样做的目的有两个：

- 现场采集时保存原始雷达和 `IMU` 数据，便于后续多次复盘
    
- 回放时重新运行 `FAST_LIO`，从同一份原始数据反复观察轨迹和建图结果
    

## 2. 默认网络与环境参数

以下参数按当前系统的默认值整理：

|项目|默认值|
|---|---|
|雷达 `IP`|`192.168.1.117`|
|开发板雷达侧 `eth0`|`192.168.1.50`|
|开发板上位机侧|`192.168.137.10`|
|上位机 `WSL2` 侧|`192.168.137.1`|
|`ROS_DOMAIN_ID`|`42`|
|`RMW_IMPLEMENTATION`|`rmw_cyclonedds_cpp`|
|开发板 `CycloneDDS` 配置文件|`~/cyclonedds_config.xml`|
|`WSL2` `CycloneDDS` 配置文件|`~/cyclonedds_config.xml`|

上述`IP`值包括后续的配置文件中设计到的`IP`值都只适合我当前使用的硬件，仅供参考，实际使用时需要根据实际情况进行调整，不要无脑抄作业

## 3. 首次配置

以下配置一般只需要完成一次。 后续如果不修改 `IP`、驱动配置或 `FAST_LIO` 参数，可以直接跳到录制和回放流程。

### 3.1 开发板修改 `Livox` 驱动配置

编辑开发板上的文件：

 ```bash
 ~/livox_ws/src/livox_ros_driver2/config/MID360s_config.json
 ```


至少确认以下字段正确：

- `host_ip`：开发板连接雷达侧网口`IP`，例如 `192.168.1.50`
    
- `lidar_configs[].ip`：雷达 `IP`，例如 `192.168.1.117`
    

参考配置（可直接复制粘贴替换原文件内容）：

```json
{  
   "lidar_summary_info": {  
     "lidar_type": 8  
   },  
   "Mid360s": {  
     "lidar_net_info": {  
       "cmd_data_port": 56100,  
       "push_msg_port": 56200,  
       "point_data_port": 56300,  
       "imu_data_port": 56400,  
       "log_data_port": 56500  
     },  
     "host_net_info": [  
       {  
         "host_ip": "192.168.1.50",  
         "cmd_data_port": 56101,  
         "push_msg_port": 56201,  
         "point_data_port": 56301,  
         "imu_data_port": 56401,  
         "log_data_port": 56501  
       }  
     ]  
   },  
   "lidar_configs": [  
     {  
       "ip": "192.168.1.117",  
       "pcl_data_type": 1,  
       "pattern_mode": 0,  
       "extrinsic_parameter": {  
         "roll": 0.0,  
         "pitch": 0.0,  
         "yaw": 0.0,  
         "x": 0,  
         "y": 0,  
         "z": 0  
       }  
     }  
   ]  
 }
```
 

### 3.2 重新编译 `Livox` 驱动

在开发板执行：

```bash
 cd ~/livox_ws  
 colcon build --symlink-install --packages-select livox_ros_driver2  
 source install/setup.bash
```


### 3.3 开发板 `CycloneDDS` 配置

开发板创建：

```bash
 ~/cyclonedds_config.xml
```


内容如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<CycloneDDS xmlns="https://cyclonedds.org/schema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="https://cyclonedds.org/schema https://cyclonedds.org/schema/cyclonedds.xsd">
    <Domain id="any">
        <General>
            <NetworkInterfaceAddress>192.168.137.10</NetworkInterfaceAddress>
            <AllowMulticast>true</AllowMulticast>
            <MaxMessageSize>65500B</MaxMessageSize>
        </General>
        <Discovery>
            <Peers>
                <Peer address="192.168.137.1"/>
            </Peers>
            <ParticipantIndex>auto</ParticipantIndex>
        </Discovery>
    </Domain>
</CycloneDDS>
```

### 3.4 `WSL2` `CycloneDDS` 配置

在 `WSL2` 创建：

```bash
 ~/cyclonedds_config.xml
```


内容如下：

 ```bash
 <?xml version="1.0" encoding="UTF-8" ?>
<CycloneDDS xmlns="https://cyclonedds.org/schema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="https://cyclonedds.org/schema https://cyclonedds.org/schema/cyclonedds.xsd">
    <Domain id="any">
        <General>
            <NetworkInterfaceAddress>192.168.137.1</NetworkInterfaceAddress>
            <AllowMulticast>true</AllowMulticast>
            <MaxMessageSize>65500B</MaxMessageSize>
        </General>
        <Discovery>
            <Peers>
                <Peer address="192.168.137.10"/>
            </Peers>
            <ParticipantIndex>auto</ParticipantIndex>
        </Discovery>
    </Domain>
</CycloneDDS>
 ```

### 3.5 如修改 `FAST_LIO` 配置，重新编译

如果你修改了：

```bash
~/fastlio_ws/src/FAST_LIO/config/mid360.yaml
```


则在开发板执行：

```bash
cd ~/fastlio_ws  
colcon build --symlink-install
```


## 4. 工具目录与脚本说明

当前录制与复盘工具目录如下：

mid360_replay_tools/  
├── README.md  
├── devboard/  
│   ├── record_mid360_bag.sh  
│   └── replay_mid360_bag.sh  
└── wsl2/  
    ├── odometry_to_path.py  
    └── start_odometry_to_path.sh

建议放置方式：

- `devboard/` 下脚本放到开发板：`~/mid360_replay_tools/devboard/`
    
- `wsl2/` 下脚本放到 `WSL2`：`~/mid360_replay_tools/wsl2/`
    

复制后执行（为对应的脚本文件添加可执行权限）：

```bash
chmod +x ~/mid360_replay_tools/devboard/*.sh  
chmod +x ~/mid360_replay_tools/wsl2/*.sh  
chmod +x ~/mid360_replay_tools/wsl2/*.py
```


## 5. 开发板录制建图过程

### 5.1 先正常启动实时建图

先按实时建图流程启动雷达驱动和 `FAST_LIO`。

开发板终端 1：

```bash
sudo ifconfig eth0 192.168.1.50 netmask 255.255.255.0 up  
ping 192.168.1.117 -c 3  
  
source /opt/ros/humble/setup.bash  
source /home/sunrise/livox_ws/install/setup.bash  
  
export ROS_DOMAIN_ID=42  
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp  
export CYCLONEDDS_URI=file:///home/sunrise/cyclonedds_config.xml  
  
ros2 launch livox_ros_driver2 msg_MID360s_launch.py
```


开发板终端 2：

```bash
source /opt/ros/humble/setup.bash  
source /home/sunrise/livox_ws/install/setup.bash  
source /home/sunrise/fastlio_ws/install/setup.bash  
  
export ROS_DOMAIN_ID=42  
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp  
export CYCLONEDDS_URI=file:///home/sunrise/cyclonedds_config.xml  
  
ros2 launch fast_lio mapping.launch.py rviz:=false
```


### 5.2 在 `WSL2` 启动 `rviz2`

在 `WSL2` 执行：

```bash
source /opt/ros/humble/setup.bash  
source ~/fastlio_ws/install/setup.bash  
  
export ROS_DOMAIN_ID=42  
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp  
export CYCLONEDDS_URI=file:///home/zhou/cyclonedds_config.xml  
  
rviz2
```


当前实际使用中，不强制要求在 `WSL2` 里先执行 `ros2 topic list` 检查。 如果 `rviz2` 已经能够收到并显示回放或实时结果，可以直接在软件里添加对应显示项。

### 5.3 在开发板开始录包

确认 `rviz2` 中建图正常后，在开发板新开终端执行：

```bash
source /opt/ros/humble/setup.bash  
source /home/sunrise/livox_ws/install/setup.bash  
source /home/sunrise/fastlio_ws/install/setup.bash  
  
export ROS_DOMAIN_ID=42  
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp  
  
~/mid360_replay_tools/devboard/record_mid360_bag.sh room_a
```


默认录制的话题：

- `/livox/lidar`（雷达建图话题）
    
- `/livox/imu`（姿态传感器话题）
    
- `/tf`
    
- `/tf_static`
    

录制目录默认形如：

```bash
~/bags/mid360/room_a_20260430_193000/
```


录制完成后，目录中包含：

- `bag/`
    
- `session_meta.txt`
    
- `bag_info.txt`
    

停止录制时在录制终端按 `Ctrl+C`。

## 6. 开发板回放录制包

### 6.1 回放前准备

建议：

- 先启动 `FAST_LIO`
    
- `WSL2` 侧先打开 `rviz2`软件
    

### 6.2 开发板启动回放

开发板先启动 `FAST_LIO`：

```bash
source /opt/ros/humble/setup.bash  
source /home/sunrise/livox_ws/install/setup.bash  
source /home/sunrise/fastlio_ws/install/setup.bash  
  
export ROS_DOMAIN_ID=42  
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp  
export CYCLONEDDS_URI=file:///home/sunrise/cyclonedds_config.xml  
  
ros2 launch fast_lio mapping.launch.py rviz:=false
```


然后在开发板新开终端执行回放：

```bash
source /opt/ros/humble/setup.bash  
source /home/sunrise/livox_ws/install/setup.bash  
source /home/sunrise/fastlio_ws/install/setup.bash  
  
export ROS_DOMAIN_ID=42  
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp  
  
~/mid360_replay_tools/devboard/replay_mid360_bag.sh ~/bags/mid360/room_a_20260430_193000
```


半速回放示例：

```bash
~/mid360_replay_tools/devboard/replay_mid360_bag.sh ~/bags/mid360/room_a_20260430_193000 0.5
```


说明：

- `1.0`：正常速度
    
- `0.5`：半速回放
    
- `2.0`：两倍速回放
    

## 7. `WSL2` 侧生成并显示完整轨迹

### 7.1 启动 `/Odometry -> /Path` 节点

在 `WSL2` 执行：

```bash
source /opt/ros/humble/setup.bash  
source ~/fastlio_ws/install/setup.bash  
  
export ROS_DOMAIN_ID=42  
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp  
export CYCLONEDDS_URI=file:///home/zhou/cyclonedds_config.xml  
  
~/mid360_replay_tools/wsl2/start_odometry_to_path.sh
```


如果你已经在当前终端手工 `source` 过上述环境，也可以直接执行：

```bash
python3 ~/mid360_replay_tools/wsl2/odometry_to_path.py
```


该节点的作用：

- 订阅 `/Odometry`
    
- 发布 `/Path`
    
- 每收到一个新位姿，就把它追加到路径中
    
- 当检测到回放时间回跳时自动清空旧轨迹
    

这意味着同一个 bag 每次重新回放时，轨迹都会从起点重新开始逐步增长。

### 7.2 在 `rviz2` 中添加显示项

在 `rviz2` 中建议至少添加：

- `PointCloud2` -> `/laser_map`
    
- `Odometry` -> `/Odometry`
    
- `Path` -> `/Path`
    

推荐设置：

- `Fixed Frame`：`camera_init`
    
- `/laser_map`：观察地图结果
    
- `/Odometry`：观察当前位姿
    
- `/Path`：观察完整历史轨迹
    

当前实际使用中，不强制要求必须先在 `WSL2` 终端看到对应话题再添加显示项。 如果 `rviz2` 已经能够在回放过程中正常显示结果，直接在软件中添加上述显示项即可。

## 8. 标准复盘顺序

推荐按以下顺序复盘：

1. 开发板不连接真实雷达
    
2. 开发板启动 `FAST_LIO`
    
3. `WSL2` 启动 `start_odometry_to_path.sh` 或直接运行 `odometry_to_path.py`
    
4. `WSL2` 打开 `rviz2`
    
5. 在 `rviz2` 中添加 `/laser_map`、`/Odometry`、`/Path`
    
6. 开发板执行 `replay_mid360_bag.sh`
    
7. 观察地图、位姿和轨迹的变化过程
    

预期效果：

- `/laser_map` 会逐步生成地图
    
- `/Odometry` 会持续更新当前位姿
    
- `/Path` 会从起点开始逐步增长，最终形成完整轨迹线
    

## 9. 常见异常处理

### 9.1 回放脚本提示 `Usage`

如果执行：

```bash
~/mid360_replay_tools/devboard/replay_mid360_bag.sh
```

直接看到 `Usage`，说明你没有传入 bag 路径参数。 正确用法示例：

```bash
~/mid360_replay_tools/devboard/replay_mid360_bag.sh ~/bags/mid360/room_a_20260430_195112
```


### 9.2 把 bag 目录当作命令执行

如果你直接输入：

```bash
~/bags/mid360/room_a_20260430_195112
```


会报：

- `是一个目录`
    

原因是你把目录本身当成了命令执行。 这个目录必须作为参数传给回放脚本，而不是单独执行。

### 9.3 `start_odometry_to_path.sh` 报 `unbound variable`

此前你遇到的：

- `AMENT_TRACE_SETUP_FILES: unbound variable`
    
- `AMENT_PYTHON_EXECUTABLE: unbound variable`
    

根因是 `ROS 2 setup.bash` 对严格的 `nounset` 模式不兼容。 当前脚本已经调整为兼容写法；如果你已经手工 `source` 完环境，也可以直接运行：

```bash
python3 ~/mid360_replay_tools/wsl2/odometry_to_path.py
```


### 9.4 回放时 `rviz2` 能显示，但 `ros2 topic list` 看不到结果

按你当前实际使用情况，这并不一定阻碍回放。 如果 `rviz2` 已经能看到回放过程，优先以软件实际显示结果为准，直接在 `rviz2` 里添加：

- `/laser_map`
    
- `/Odometry`
    
- `/Path`
    

### 9.5 `CycloneDDS` 提示 `NetworkInterfaceAddress deprecated`

这只是配置项弃用警告，不是当前录制或回放失败的直接原因。 只要实际通信和显示正常，可以先忽略该警告。

## 10. 复盘结果判断重点

每次复盘重点观察：

- 轨迹是否连续
    
- 转弯处是否出现明显突跳
    
- 地图边界是否撕裂
    
- 停止移动后地图是否稳定
    
- 同一 bag 在不同参数下轨迹和地图是否改善
    

## 11. 可选环境变量

录制脚本支持以下环境变量：

- `MID360_BAG_ROOT`：录制根目录，默认 `~/bags/mid360`
    
- `LIDAR_IP`：写入元信息，默认 `192.168.1.117`
    
- `LIDAR_HOST_IP`：写入元信息，默认 `192.168.1.50`
    
- `UPLINK_IP`：写入元信息，默认 `192.168.137.10`
    
- `FASTLIO_CONFIG`：写入元信息，默认 `~/fastlio_ws/src/FAST_LIO/config/mid360.yaml`
    

示例：

```bash
 export MID360_BAG_ROOT=~/bags/mid360  
 export LIDAR_IP=192.168.1.117  
 export LIDAR_HOST_IP=192.168.1.50  
 export UPLINK_IP=192.168.137.10
```
