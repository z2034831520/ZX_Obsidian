---
title: Bootloader与固件升级
aliases:
  - Bootloader 与固件升级
type: knowledge
tags: [单片机, STM32, Bootloader, 固件升级]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# Bootloader 与固件升级

## 两类 Bootloader

STM32 芯片通常包含厂商固化的系统存储器 Bootloader，可通过特定启动配置进入，并使用芯片支持的 UART、USB、CAN、I2C 或 SPI 等接口下载。支持接口和引脚因型号而异，应查询 AN2606。

用户 Bootloader 位于 Flash 起始区域，负责校验、升级和跳转到应用程序。应用程序需要放在约定的偏移地址，并调整链接脚本和中断向量表位置。

## 基本升级流程

1. 接收固件头和镜像数据。
2. 检查型号、版本、长度和完整性。
3. 擦除目标区域并分块写入。
4. 回读或计算 CRC/哈希。
5. 设置“镜像有效”标志。
6. 重启并跳转到应用。
7. 启动失败时回滚或保留恢复入口。

## 跳转应用的关键步骤

跳转前通常需要停止中断和 SysTick、复位外设状态、设置 MSP、更新 VTOR，并跳到应用复位向量。不同内核和芯片要求不同，不能直接复制未经验证的跳转代码。

## 安全性

完整性校验只能发现损坏，不能证明来源可信。需要防止恶意固件时，应加入数字签名、公钥校验、防回滚计数和密钥保护。

## 常见问题

- Bootloader 和应用 Flash 区域重叠。
- 应用链接地址或 VTOR 未调整。
- 写 Flash 时电源中断导致设备失效。
- 只做 CRC，不做身份认证。
- 升级前未验证目标型号和固件长度。

## 参考资料

- [STM32 系统存储器 Boot 模式 AN2606](https://www.st.com/resource/en/application_note/an2606-stm32-microcontroller-system-memory-boot-mode-stmicroelectronics.pdf)
