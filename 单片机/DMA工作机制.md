---
title: DMA工作机制
aliases:
  - DMA 工作机制
type: knowledge
tags: [单片机, STM32, DMA]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# DMA 工作机制

DMA 在外设和存储器之间搬运数据，CPU 只负责配置和处理完成事件，从而降低逐字节中断开销。

## 核心配置

- 源地址与目标地址。
- 数据方向：外设到内存、内存到外设或内存到内存。
- 单次传输长度。
- 源/目标地址是否递增。
- 数据宽度。
- 普通或循环模式。
- 通道、请求或 DMAMUX 映射。
- 完成、半完成和错误中断。

## 典型模式

UART 接收常使用“DMA 循环缓冲区 + 空闲中断”识别不定长帧；ADC 连续采样常使用循环 DMA，并在半完成和完成回调中分块处理。

## 缓存一致性

带数据缓存的 Cortex-M7/M33 等平台上，DMA 与 CPU 可能看到不同内容。发送前可能需要清理 Cache，接收后可能需要失效 Cache，并保证缓冲区按缓存行对齐。具体操作必须结合芯片架构和 MPU 配置。

## 常见问题

- DMA 请求映射错误。
- 缓冲区是已经失效的局部变量。
- 数据宽度和地址递增配置不匹配。
- 未清除错误标志或未重新启动普通模式。
- CPU 与 DMA 同时修改同一缓冲区。
- 忽略缓存一致性。

## 参考资料

- [STM32CubeMX 外设与 DMA 配置](https://dev.st.com/stm32cube-docs/stm32cubemx2/1.0.0/en/docs/markup/CubeMX2_UserManual/CubeMX2_UserManual_Peripherals.html)
