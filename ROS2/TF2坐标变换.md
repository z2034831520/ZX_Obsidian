---
title: TF2坐标变换
aliases: [TF2 坐标变换]
type: knowledge
tags: [ROS2, TF2, 坐标系]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# TF2 坐标变换

TF2 维护带时间戳的坐标变换树，使节点可以查询两个坐标系在某一时刻的相对位姿。

## 基本概念

- **父坐标系与子坐标系**：一条变换描述子坐标系在父坐标系中的位姿。
- **动态变换**：随时间变化，通过 `/tf` 发布。
- **静态变换**：固定安装关系，通过 `/tf_static` 发布。
- **Buffer 与 Listener**：缓存变换并提供查询。
- **Broadcaster**：发布变换。

常见机器人链路：

```text
map → odom → base_link → sensor_link
```

TF 树不能有多个父节点或闭环。帧名称、单位和坐标约定必须统一。

## 时间

查询“当前”变换时，传感器消息可能对应更早的采样时刻。正确做法是使用消息时间戳查询对应变换，并为通信延迟保留合理缓存。查询未来尚未发布的变换会触发 extrapolation 错误。

## 排查工具

```bash
ros2 run tf2_ros tf2_echo base_link sensor_link
ros2 run tf2_tools view_frames
ros2 topic echo /tf
```

## 常见问题

- 父子坐标系方向写反。
- 静态安装关系被高频重复发布。
- 使用当前时间转换旧传感器数据。
- NED、ENU 或机体系轴定义混用。
- 两个节点同时发布同一子坐标系。

## 参考资料

- [ROS 2 TF2 tutorials](https://docs.ros.org/en/rolling/Tutorials/Intermediate/Tf2/Tf2-Main.html)
