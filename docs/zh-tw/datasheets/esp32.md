# ESP32 GPIO 參考

## Pin Aliases

| alias       | pin |
|-------------|-----|
| builtin_led | 2   |
| red_led     | 2   |

## 常用 Pin（ESP32 / ESP32-C3）

- **GPIO 2**：多數開發板的板載 LED（output）
- **GPIO 13**：通用輸出
- **GPIO 21/20**：通常用於 UART0 TX/RX（使用 serial 時避免使用）

## 協定

ZeroClaw host 透過 serial（115200 baud）傳送 JSON：
- `gpio_read`：`{"id":"1","cmd":"gpio_read","args":{"pin":13}}`
- `gpio_write`：`{"id":"1","cmd":"gpio_write","args":{"pin":13,"value":1}}`

回應：`{"id":"1","ok":true,"result":"0"}` 或 `{"id":"1","ok":true,"result":"done"}`
