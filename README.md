# 基于 STM32 的可穿戴老人防跌倒检测与双模远程告警系统
> **An Ultra-Low-Power Wearable Fall Detection & Dual-Mode Alert System based on STM32**

[![Platform](https://img.shields.io/badge/Platform-STM32F103C8T6-blue.svg)](https://www.st.com/)
[![RTOS](https://img.shields.io/badge/RTOS-FreeRTOS-green.svg)](https://www.freertos.org/)
[![Language](https://img.shields.io/badge/Language-C-orange.svg)](https://en.wikipedia.org/wiki/C_(programming_language))
[![Communication](https://img.shields.io/badge/Comms-NB--IoT%20%7C%20LoRa-purple.svg)]()
[![Standby Power](https://img.shields.io/badge/Standby%20Current-%3C%2050%CE%BCA-red.svg)]()
[![License](https://img.shields.io/badge/License-MIT-brightgreen.svg)](LICENSE)

---

## 项目简介 (Overview)

随着老龄化社会的加剧，独居老人在发生意外跌倒时若得不到及时救助，极易造成严重后果。本项目设计并实现了一套具备**高精度实时监测、多级姿态识别、卫星精准定位以及远程冗余告警**功能的穿戴式物联网终端原型。

主控采用 **STM32F103C8T6**，搭载 **FreeRTOS** 实时操作系统。针对偏远及弱网环境，创新性地构建了 **NB-IoT + LoRa 双模容灾通信架构**；同时通过深入芯片物理层的电源状态机设计，将整机待机休眠电流控制在 **50μA 以下**，大幅延长了可穿戴设备的电池续航时间。

---

## 核心特性与技术亮点 (Key Highlights)

-  **高精准多级跌倒识别算法**：结合六轴传感器（MPU6050），通过**四元数姿态解算**与**卡尔曼滤波（Kalman Filter）**降噪，设计“合加速度突变 ➡️ 姿态倾角突变 ➡️ 静止时间窗”三阶判定逻辑，彻底解决日常生活弯腰、坐下等动作带来的误判。
-  **极致超低功耗电源管理 (< 50μA)**：深入分析 MCU 功耗模型，借助内部 RTC 定时唤醒机制，设计 **Run / Sleep / Stop** 动态电源状态机，在休眠时动态关断外部传感器与无线通信模块供电（MOS管电源旁路）。
-  **双模无线通信冗余链路（容错机制）**：
  - **主力链路**：基于 NB-IoT（通过 AT 指令以 TCP 协议直连云端），完成日常数据上传与急救告警。
  - **备用链路**：在偏远无基站信号或主链路超时时，自动触发状态机无缝切换至 **LoRa 扩频通信**（点对点直传本地网关），确保报警帧必达。
---

##  系统架构与硬件选型 (Hardware Architecture)

### 1. 硬件拓扑架构
```text
           +-----------------------------------------------+
           |               STM32F103C8T6                   |
           |       (ARM Cortex-M3 @ 72MHz, FreeRTOS)       |
           +-----------------------+-----------------------+
                                   |
         +-----------------+-------+-------+-----------------+
         | I2C (DMA)       | UART2 (DMA)   | SPI1            | UART3 (DMA)
         v                 v               v                 v
   +-----------+     +-----------+   +-----------+     +-----------+
   |  MPU6050  |     |  AT6558R  |   |  SX1278   |     |  NB-IoT   |
   | (6-Axis)  |     | (GPS 定位)|   | (LoRa 备用)|    | (TCP 主链)|
   +-----------+     +-----------+   +-----------+     +-----------+# 基于 STM32 的可穿戴老人防跌倒检测与双模远程告警系统
> **An Ultra-Low-Power Wearable Fall Detection & Dual-Mode Alert System based on STM32 & FreeRTOS**
2. 核心物料清单 (BOM)
器件类型	芯片型号	通信接口	作用功能
主控芯片	STM32F103C8T6	-	72MHz 主频，负责算法、RTOS 调度与电源管理
惯性传感器	MPU6050	I2C (支持硬件 DMA)	三轴加速度计 + 三轴陀螺仪，采集姿态角速度与加速度
定位芯片	AT6558R	UART (串口空闲中断)	获取经纬度定位信息，提供救援坐标
主通信模块	移远 BC26/BC20 等	UART (AT 指令)	NB-IoT 上云，日常低功耗数据同步与报警
备用通信	SX1278 (LoRa)	SPI	433MHz/470MHz 远距离扩频穿透通信，弱网容灾备用
计步芯片	DS3553	I2C / INT	独立低功耗日常健康计步管理
3. 引脚映射表 (Pinout)
外设模块	外设引脚	MCU 引脚	模式配置 / 描述
MPU6050	SCL / SDA	PB6 / PB7	I2C1 开漏上拉输出，400kHz 快速模式
INT (中断)	PA0	EXTI0 外部中断输入，用于静止被动唤醒
AT6558R	TX / RX	PA3 / PA2	USART2 (DMA 自动接收变长 NMEA 语句)
NB-IoT	TX / RX	PB11 / PB10	USART3 (空闲中断 IDLE + DMA 传输)
LoRa (SX1278)	SCK/MISO/MOSI	PA5 / PA6 / PA7	SPI1 全双工主机模式
CS / DIO0	PA4 / PB0	片选与中断响应引脚
电源控制管	PWR_KEY	PB1	GPIO 推挽输出，控制高侧 P-MOS 截止实现彻底断电


---
