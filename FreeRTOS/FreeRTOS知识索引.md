---
title: FreeRTOS知识索引
type: index
tags:
  - 知识库
  - FreeRTOS
  - 嵌入式
status: active
created: 2026-08-08
updated: 2026-08-08
---

# FreeRTOS知识索引

> FreeRTOS 学习与实践的二级入口。建议按照“基础概念 → 调度机制 → 系统服务 → 工程实践”的顺序阅读。

## 推荐学习路线

1. [[FreeRTOS/FreeRTOS介绍]]
2. [[FreeRTOS/FreeRTOS命名规则]]
3. [[FreeRTOS/任务状态]]
4. [[FreeRTOS/抢占式任务调度]]
5. [[FreeRTOS/定时器服务任务]]
6. [[FreeRTOS/FreeRTOS底层实现原理]]

## 基础概念

- [[FreeRTOS/FreeRTOS介绍]]：FreeRTOS 的定位、用途以及与裸机多任务思想的关系。
- [[FreeRTOS/FreeRTOS命名规则]]：内核对象、函数和数据类型的常见命名方式。
- [[FreeRTOS/任务状态]]：任务运行、就绪、阻塞和挂起等状态。

## 调度与底层机制

- [[FreeRTOS/抢占式任务调度]]：抢占式调度的基本工作方式。
- [[FreeRTOS/定时器服务任务]]：软件定时器及其服务任务。
- [[FreeRTOS/FreeRTOS底层实现原理]]：内核调度与底层实现思路。

## 开发环境与工程实践

- [[FreeRTOS/CLion开发FreeRTOS的基本配置]]：使用 CLion 开发 FreeRTOS 工程的基础配置。
- [[FreeRTOS/CLion使用教程——FreeRTOS]]：面向 FreeRTOS 工程的 CLion 使用流程。
- [[FreeRTOS/OLED中文字符显示——基于CMake工程]]：CMake 工程中的 OLED 中文显示实践。

## 相关领域

- [[单片机/时间片轮转]]
- [[单片机/伪调度器架构]]
- [[单片机/状态机]]

## 待补充
- [x] [[FreeRTOS/任务创建、删除与优先级]]
- [x] [[FreeRTOS/队列、信号量和互斥锁]]
- [x] [[FreeRTOS/事件组与任务通知]]
- [x] [[FreeRTOS/内存管理与栈溢出检测]]
- [x] [[FreeRTOS/中断与FreeRTOS API使用边界|中断与 FreeRTOS API 使用边界]]
