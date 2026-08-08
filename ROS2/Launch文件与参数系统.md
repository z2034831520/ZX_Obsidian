---
title: Launch文件与参数系统
aliases: [Launch 文件与参数系统]
type: knowledge
tags: [ROS2, Launch, 参数]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# Launch 文件与参数系统

## Launch

Launch 用于一次启动多个节点、设置命名空间、重映射话题、传递参数和组织启动条件。Python Launch 文件通常返回 `LaunchDescription`。

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package="demo_nodes_cpp",
            executable="talker",
            name="talker",
            parameters=[{"rate": 10.0}],
            remappings=[("chatter", "status")],
            output="screen",
        )
    ])
```

Launch 文件必须作为包资源安装，才能通过 `ros2 launch package file.launch.py` 找到。

## 参数系统

参数属于具体节点，由“名称—类型—值”组成。节点通常先声明参数，再读取或注册动态修改回调。

```bash
ros2 param list
ros2 param get /node_name rate
ros2 param set /node_name rate 20.0
ros2 param dump /node_name
```

参数文件使用 YAML，并按节点名称和 `ros__parameters` 组织。节点名或命名空间不匹配时，参数可能不会生效。

## 参数与话题的边界

参数适合低频配置，不适合持续高速数据；连续数据应使用话题，触发操作可使用服务或动作。

## 常见问题

- 参数未声明就读取。
- YAML 层级、节点名或数据类型错误。
- Launch 文件未安装到 share 目录。
- 用参数传递实时传感器数据。
