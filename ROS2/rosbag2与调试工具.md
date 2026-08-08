---
title: rosbag2与调试工具
aliases: [rosbag2 与调试工具]
type: knowledge
tags: [ROS2, rosbag2, 调试]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# rosbag2 与调试工具

## rosbag2

rosbag2 用于记录和回放话题数据，适合复现传感器输入、算法离线验证和故障证据保存。

```bash
ros2 bag record /imu/data /odom /tf /tf_static
ros2 bag record -a
ros2 bag info my_bag
ros2 bag play my_bag
```

记录前确认磁盘空间、目标话题频率、消息大小和 QoS。高带宽数据可能需要调整存储、缓存和压缩策略。回放时注意仿真时间 `/clock`、播放速率和 QoS 覆盖。

## 常用调试工具

```bash
ros2 node list
ros2 node info /node
ros2 topic list
ros2 topic echo /topic
ros2 topic hz /topic
ros2 topic info /topic --verbose
ros2 service list
ros2 action list
ros2 interface show package/msg/Type
ros2 doctor --report
```

图形工具包括 `rqt_graph`、`rqt_console`、RViz2 和 `rqt_bag`。

## 推荐故障定位顺序

1. 检查环境和目标 ROS_DOMAIN_ID。
2. 检查节点是否存在、是否持续运行。
3. 检查话题名称、类型和端点数量。
4. 检查 QoS 是否兼容。
5. 检查消息频率、时间戳、frame_id 和 TF。
6. 录制最小 rosbag，在离线环境复现。
7. 保存版本、参数、Launch 配置和日志。

## 参考资料

- [rosbag2 官方文档](https://docs.ros.org/en/rolling/p/rosbag2/index.html)
