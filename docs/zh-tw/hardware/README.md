# 硬體和週邊文件

開發板整合、韌體和週邊裝置。

ZeroClaw 的硬體系統透過 `Peripheral` trait 實現直接控制微控制器和週邊裝置。每個開發板提供 GPIO、ADC 和感測器操作的 tool，在 STM32 Nucleo、Raspberry Pi 和 ESP32 等開發板上實現 agent 驅動的硬體互動。參閱 [../hardware-peripherals-design.md](../hardware-peripherals-design.md) 了解完整架構。

## 起始點

- 架構和週邊模型：[../hardware-peripherals-design.md](../hardware-peripherals-design.md)
- 新增開發板/tool：[../adding-boards-and-tools.md](../adding-boards-and-tools.md)
- Nucleo 設定：[../nucleo-setup.md](../nucleo-setup.md)
- Arduino Uno Q 設定：[../arduino-uno-q-setup.md](../arduino-uno-q-setup.md)

## Datasheets

- Datasheet 索引：[../datasheets](../datasheets)
- STM32 Nucleo-F401RE：[../datasheets/nucleo-f401re.md](../datasheets/nucleo-f401re.md)
- Arduino Uno：[../datasheets/arduino-uno.md](../datasheets/arduino-uno.md)
- ESP32：[../datasheets/esp32.md](../datasheets/esp32.md)
