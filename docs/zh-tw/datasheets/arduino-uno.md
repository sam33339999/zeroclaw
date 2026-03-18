# Arduino Uno

## Pin Aliases

| alias       | pin |
|-------------|-----|
| red_led     | 13  |
| builtin_led | 13  |
| user_led    | 13  |

## 概覽

Arduino Uno 是基於 ATmega328P 的微控制器開發板。有 14 個數位 I/O pin（0–13）和 6 個類比輸入（A0–A5）。

## 數位 Pin

- **Pins 0–13：** 數位 I/O。可設為 INPUT 或 OUTPUT。
- **Pin 13：** 板載 LED（onboard）。連接 LED 到 GND 或用於輸出信號。
- **Pins 0–1：** 也用於 Serial（RX/TX）。使用 Serial 時避免使用。

## GPIO

- `digitalWrite(pin, HIGH)` 或 `digitalWrite(pin, LOW)` 輸出信號。
- `digitalRead(pin)` 讀取輸入（回傳 0 或 1）。
- ZeroClaw 協定中的 pin 編號：0–13。

## Serial

- UART 在 pin 0（RX）和 1（TX）。
- USB 透過 ATmega16U2 或 CH340（clone 版本）。
- Baud rate：ZeroClaw 韌體使用 115200。

## ZeroClaw Tools

- `gpio_read`：讀取 pin 值（0 或 1）。
- `gpio_write`：設定 pin 為高（1）或低（0）。
- `arduino_upload`：Agent 生成完整的 Arduino sketch 程式碼；ZeroClaw 透過 arduino-cli 編譯並上傳。Pin 13 = 板載 LED。
