# `ROS`小车操作手册

## ROS教程网址：https://gitee.com/gwmunan/ros2/wikis/pages

## 命令总结

### 统一命令

#### 一键安装`ros2`环境命令

```bash
wget http://fishros.com/install -O fishros && . fishros
```

#### 统一接口

此命令在小车的终端以及虚拟机终端都要执行

```bash
export ROS_DOMAIN_ID=30
```

### 树莓派端

#### 安装`TurtleBot3` 全家桶

```bash
sudo apt update
sudo apt install ros-humble-turtlebot3* -y
```

#### 配置小车基本信息（永久生效）

```bash 
echo "export TURTLEBOT3_MODEL=burger" >> ~/.bashrc
source ~/.bashrc
```

#### 配置雷达模块信息（永久生效）

```bash
echo "export LDS_MODEL=LDS-01" >> ~/.bashrc
source ~/.bashrc
```

#### 启动所有基本外设（不包括摄像头 ）

```bash
ros2 launch turtlebot3_bringup robot.launch.py
```

#### 开启键盘控制

```bash 
export ROS_DOMAIN_ID=30
export TURTLEBOT3_MODEL=burger
ros2 run turtlebot3_teleop teleop_keyboard
```



#### 启动摄像头模块命令

命令最后的`[320,240]`，代表图像分辨率，可以根据实际情况进行调整

```bash
ros2 run v4l2_camera v4l2_camera_node --ros-args -p image_size:="[320,240]"
```

### 虚拟机端

#### `WSL`网络配置

如果使用的是`WSL`子系统作为上位机，那么需要修改对应的网络模式，默认情况下`wsl`是`NET`模式，一般在启动时会报警告或者提醒你当前的网络模式，内容如下

```bash
Microsoft Windows [版本 10.0.26200.8037]
(c) Microsoft Corporation。保留所有权利。

C:\Users\zhou>ros2 topic list
'ros2' 不是内部或外部命令，也不是可运行的程序
或批处理文件。

C:\Users\zhou>wsl
wsl: 检测到 localhost 代理配置，但未镜像到 WSL。NAT 模式下的 WSL 不支持 localhost 代理。
zhou@LAPTOP-8EA68M0L:/mnt/c/Users/zhou$ ros2 topic list
```

此时就需要修改对应的网络模式，修改时需要在`CMD`中运行以下命令

```bash
echo [wsl2] > %USERPROFILE%\.wslconfig
echo networkingMode=mirrored >> %USERPROFILE%\.wslconfig
echo firewall=true >> %USERPROFILE%\.wslconfig
wsl --shutdown
```



#### 环境变量配置

安装完成后需要通知虚拟机每次开机时自动加载`ROS2`

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
echo "export ROS_DOMAIN_ID=30" >> ~/.bashrc
source ~/.bashrc
```

安装好环境后输入以下命令进行测试，如果出现海龟则说明配置成功

```bash 
ros2 run turtlesim turtlesim_node
```



#### Windows终端快捷键

**`Alt + Shift + 加号 (+)`** ：左右垂直分屏。

**`Alt + Shift + 减号 (-)`** ：上下水平分屏。

**`Alt + 方向键`** ：在不同的分屏窗口之间移动光标。

**`Ctrl + Shift + W`** ：关闭当前光标所在的分屏

#### 安装专属扩展包

安装好`ros2`环境之后并不能直接运行`ros2`的相关命令，需要运行以下指令安装对应的扩展包

```bash
sudo apt install ros-humble-turtlebot3* -y
```

#### 上位机查看当前话题数量

```bash
# 图形化界面查看
rqt	# 前提是要与小车端统一端口
# 命令行中查看
ros2 topic list
```



#### 键盘驱动

命令：

```bash
export ROS_DOMAIN_ID=30
export TURTLEBOT3_MODEL=burger
ros2 run turtlebot3_teleop teleop_keyboard
```



#### 手柄启动

安装驱动

```bash
sudo apt update
sudo apt install ros-humble-joy ros-humble-teleop-twist-joy -y
```

驱动手柄

```bash
ros2 launch teleop_twist_joy teleop-launch.py joy_config:='xbox'
```

#### 启动`3D`模拟

注入变量库、声明小车型号

```bash
# 1. 手动注入 Gazebo 的核心环境变量库（解决 px != 0 的终极解药）
source /usr/share/gazebo/setup.sh

# 2. 声明你的小车模型
export TURTLEBOT3_MODEL=burger
```

启动`3D`环境

```bash
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```

#### 开启2维建图

```bash 
export ROS_DOMAIN_ID=30
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_cartographer cartographer.launch.py
```

#### 保存建图结果

```bash
export ROS_DOMAIN_ID=30
# ~/就是建图结果的最终保存路径，可以sui'yi
ros2 run nav2_map_server map_saver_cli -f ~/my_room_map
```

#### 远程连接小车（SSH）

连接之前试试能不能`ping`通小车的`IP`地址

```bash
ssh <用户名>@<小车的IP地址>
```

通过`ssh`远程连接小车之后就可以直接运行树莓派端的命令

#### 多机协同启动小车命令

```bash
ros2 launch ./multi_robot_bringup.launch.py bot_name:=tb3	# tb3代表小车的代号，可以根据实际情况进行更改
```

#### 键盘控制对应小车命令

```bash
ros2 run turtlebot3_teleop teleop_keyboard --ros-args -r /cmd_vel:=/<小车域名>/cmd_vel
```

#### 多机协同启动文件内容

```python
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, GroupAction, IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import LaunchConfiguration
from launch_ros.actions import PushRosNamespace, SetRemap

def generate_launch_description():
    turtlebot3_bringup_dir = get_package_share_directory('turtlebot3_bringup')

    # 1. 核心改动：将变量名改为 'bot_name'，防止与官方底层参数冲突！
    bot_name_arg = DeclareLaunchArgument(
        'bot_name',
        default_value='tb1',
        description='Namespace for the TurtleBot3'
    )
    bot_name = LaunchConfiguration('bot_name')

    bringup_group = GroupAction([
        # 2. 干净利落地推一层命名空间
        PushRosNamespace(bot_name),

        # 3. 仅保留对 TF 坐标树的全局映射（防止 RViz 报错）
        SetRemap(src='/tf', dst='/tf'),
        SetRemap(src='/tf_static', dst='/tf_static'),

        # 4. 启动官方驱动
        IncludeLaunchDescription(
            PythonLaunchDescriptionSource(
                os.path.join(turtlebot3_bringup_dir, 'launch', 'robot.launch.py')
            )
        )
    ])

    return LaunchDescription([
        bot_name_arg,
        bringup_group
    ])
```

#### 写入方法

```bash 
cat << 'EOF' > multi_robot_bringup.launch.py
```

运行当前指令后直接粘贴文件内容，然后回车输入`EOF`，最后再次回车

#### 编队启动流程

```bash
./para_follow.py --ros-args -p leader:=tb1 -p follower:=tb2
```
