# ESP32

# 一、简介

## 1、GPIO 针脚

![GPIO of ESP32](../assets/ESP32-GPIO.jpg)

![uPesy ESP32 Wroom Low Power DevKit](https://www.upesy.com/cdn/shop/products/upesy-esp32-wroom-low-power-devkit-pinout.png?v=1679563719&width=1445)

![img](https://docs.sunfounder.com/projects/esp32-starter-kit/ja/latest/_images/esp32_pinout.jpg)

有 28 个 0～16位PWM通道。带有‘~’符号的引脚表示支持 PWM



| 芯片型号     | 核心        | Wi-Fi           | 蓝牙             | 特点            | 常见用途 |
| ------------ | ----------- | --------------- | ---------------- | --------------- | -------- |
| **ESP32**    | 双核 Xtensa | 2.4G            | BT Classic + BLE | 经典款，资源多  | 老项目   |
| **ESP32-S2** | 单核        | 2.4G            | ❌                | USB OTG         | USB 设备 |
| **ESP32-S3** | 双核        | 2.4G            | BLE              | AI/USB          | 新主流   |
| **ESP32-C3** | 单核 RISC-V | 2.4G            | BLE              | 低功耗、便宜    | IoT      |
| **ESP32-C6** | 单核 RISC-V | 2.4G + 802.15.4 | BLE              | Thread / Matter | HA       |
| **ESP32-H2** | 单核        | ❌               | BLE + 802.15.4   | 无 Wi-Fi        | Zigbee   |

# 二、

# 三、IDE

## 1、VSCode

- vscode安装插件platformio
- brew安装platformio

编译

```bash
pio run
```

上传

```bash
pio run -t upload
```

监视串口

```bash
pio device monitor
```
