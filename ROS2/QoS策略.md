---
title: QoS策略
aliases: [QoS 策略]
type: knowledge
tags: [ROS2, QoS, DDS, 通信]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# QoS 策略

ROS 2 基于 DDS，发布者和订阅者需要兼容的 QoS 才能通信。

## 主要策略

| 策略 | 常见选项 | 含义 |
|---|---|---|
| Reliability | Reliable / Best Effort | 可靠重传或尽力发送 |
| Durability | Volatile / Transient Local | 是否向后加入者保留历史样本 |
| History | Keep Last / Keep All | 保存有限或全部历史 |
| Depth | 数值 | Keep Last 队列深度 |
| Deadline | 时间 | 期望消息更新周期 |
| Lifespan | 时间 | 消息有效时长 |
| Liveliness | 策略与租约 | 判断发布者是否仍存活 |

传感器高频数据通常更关心新鲜度，可使用 Sensor Data QoS；配置、地图等数据可能更强调可靠性或 transient local。

## 兼容性

QoS 不是“双方完全相同才通信”，而是请求与提供必须兼容。典型问题是订阅者请求 Reliable，但发布者只提供 Best Effort。排查时应检查端点信息：

```bash
ros2 topic info /scan --verbose
```

## 选择原则

- 有线稳定网络、命令或低频关键数据：倾向 Reliable。
- 高频传感器、允许丢少量旧帧：倾向 Best Effort。
- 后启动节点需要立即得到最后状态：考虑 Transient Local。
- 队列深度根据频率、处理时延和内存共同确定。

## 常见问题

- 只看话题名称相同，不检查 QoS。
- 队列过深导致处理旧数据。
- 为所有数据一律使用 Reliable，弱网络下延迟堆积。
- rosbag2 记录或回放时 QoS 不兼容。

## 参考资料

- [ROS 2 QoS concepts](https://docs.ros.org/en/rolling/Concepts/Intermediate/About-Quality-of-Service-Settings.html)
