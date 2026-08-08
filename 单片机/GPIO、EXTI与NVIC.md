---
title: GPIO、EXTI与NVIC
aliases:
  - GPIO、EXTI 与 NVIC
type: knowledge
tags: [单片机, STM32, GPIO, EXTI, NVIC]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# GPIO、EXTI 与 NVIC

## 三者关系

GPIO 决定引脚的电气模式，EXTI 负责检测外部或内部事件，NVIC 负责管理进入 CPU 的中断请求。典型链路是：

```text
引脚电平变化 → GPIO输入 → EXTI边沿检测 → NVIC仲裁 → IRQHandler → HAL回调
```

## GPIO

常用模式包括输入、推挽输出、开漏输出、复用功能和模拟模式。输入还需选择上拉、下拉或浮空；输出速度影响边沿和电磁干扰，并非越高越好。开漏总线通常需要外部上拉。

## EXTI

STM32 的 GPIO 外部中断通常按引脚号映射到 EXTI 线，例如 PA0、PB0 竞争 EXTI0。触发方式可设为上升沿、下降沿或双边沿。中断处理函数必须及时清除挂起标志，否则会反复进入。

## NVIC

NVIC 控制中断使能、抢占优先级和响应优先级。优先级数值越小通常越紧急，但实际有效位数取决于芯片。使用 FreeRTOS 时，还要满足可调用 RTOS API 的中断优先级边界。

## 排查顺序

1. 检查实际引脚和复用映射。
2. 检查 GPIO 时钟与输入电平。
3. 检查 EXTI 线、边沿和挂起位。
4. 检查 NVIC 使能与优先级。
5. 检查 IRQHandler 和回调函数是否真实到达。

## 参考资料

- [STM32CubeMX2 EXTI 配置](https://dev.st.com/stm32cube-docs/stm32cubemx2/1.1.0/en/docs/markup/CubeMX2_How_To/CubeMX2_How_To_Configure_EXTI.html)
