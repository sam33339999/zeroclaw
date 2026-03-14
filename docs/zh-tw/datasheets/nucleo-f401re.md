# Nucleo-F401RE GPIO

## Pin Aliases

| alias       | pin |
|-------------|-----|
| red_led     | 13  |
| user_led    | 13  |
| ld2         | 13  |
| builtin_led | 13  |

## GPIO

Pin 13：User LED（LD2）
- Output，主動高（active high）
- PA5 在 STM32F401 上

## 協定

ZeroClaw host 透過 serial（USART2，115200 baud）傳送 JSON：
- `gpio_read`：`{"id":"1","cmd":"gpio_read","args":{"pin":13}}`
- `gpio_write`：`{"id":"1","cmd":"gpio_write","args":{"pin":13,"value":1}}`

回應：`{"id":"1","ok":true,"result":"0"}` 或 `{"id":"1","ok":true,"result":"done"}`
