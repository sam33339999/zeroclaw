# 入門文件

適用於首次安裝和快速上手。

## 入門路線圖

1. 概覽和快速啟動：[../../../README.zh-CN.md](../../../README.md)
2. 一鍵安裝和雙重 bootstrap 模式：[../one-click-bootstrap.md](../one-click-bootstrap.md)
3. 完整安裝指南：[../full-setup.md](../full-setup.md)
4. 依任務查詢指令：[../commands-reference.md](../commands-reference.md)

## 選擇方向

| 情境 | 指令 |
|----------|---------|
| 有 API key，想最快安裝 | `zeroclaw onboard --api-key sk-... --provider openrouter` |
| 想要逐步引導 | `zeroclaw onboard --interactive` |
| 已有設定，只需修改頻道 | `zeroclaw onboard --channels-only` |

## 設定和驗證

- 快速設定：`zeroclaw onboard --api-key "sk-..." --provider openrouter`
- 互動式設定：`zeroclaw onboard --interactive`
- 環境檢查：`zeroclaw status` + `zeroclaw doctor`

## 後續步驟

- Runtime 運維：[../operations/README.md](../operations/README.md)
- 參考查詢：[../reference/README.md](../reference/README.md)
