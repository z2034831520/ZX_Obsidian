---
title: 中断与FreeRTOS API使用边界
aliases:
  - 中断与 FreeRTOS API 使用边界
type: knowledge
tags: [FreeRTOS, 中断, ISR]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# 中断与 FreeRTOS API 使用边界

## 基本原则

中断服务函数应尽可能短，只完成读取状态、清除标志、搬运少量数据和唤醒任务。复杂协议解析、日志输出和长时间计算应交给任务执行。

中断中不能调用会阻塞的普通任务 API，只能使用明确以 `FromISR` 结尾的接口，例如：

- `xQueueSendFromISR()`
- `xSemaphoreGiveFromISR()`
- `vTaskNotifyGiveFromISR()`

```c
BaseType_t higher_woken = pdFALSE;

vTaskNotifyGiveFromISR(task_handle, &higher_woken);
portYIELD_FROM_ISR(higher_woken);
```

如果唤醒了比当前任务优先级更高的任务，应通过移植层宏请求在中断退出时切换任务。

## 中断优先级

在 Cortex-M 中，只有满足 FreeRTOS 可管理优先级范围的中断才能调用 `FromISR` API。具体边界由 `configPRIO_BITS`、`configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY` 或 `configMAX_SYSCALL_INTERRUPT_PRIORITY` 等配置决定。数值与实际紧急程度的关系容易混淆，必须结合芯片 NVIC 位宽和工程配置检查。

## 禁止事项

- 在 ISR 中调用 `vTaskDelay()`、普通队列发送或带阻塞时间的 API。
- 在 ISR 中使用不确定耗时的 `printf`、动态内存分配或复杂协议解析。
- 忘记清除外设中断标志，造成中断风暴。
- 将可调用 FreeRTOS API 的中断配置为过高紧急级别。

## 参考资料

- [FreeRTOS Queues and ISR API boundary](https://www.freertos.org/Documentation/02-Kernel/02-Kernel-features/02-Queues-mutexes-and-semaphores/01-Queues)
