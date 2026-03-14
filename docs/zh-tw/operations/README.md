# 運維和部署文件

適用於持續執行或在生產環境中操作 ZeroClaw 的 operator。

## 核心運維

- Day-2 手冊：[../operations-runbook.md](../operations-runbook.md)
- Release 手冊：[../release-process.md](../release-process.md)
- 疑難排解矩陣：[../troubleshooting.md](../troubleshooting.md)
- 網路/gateway 安全部署：[../network-deployment.md](../network-deployment.md)
- Mattermost 設定（頻道專用）：[../mattermost-setup.md](../mattermost-setup.md)

## 常見流程

1. 驗證 runtime（`status`、`doctor`、`channel doctor`）
2. 每次套用一個設定變更
3. 重啟 service/daemon
4. 驗證頻道和 gateway 健康狀態
5. 若行為退化則快速 rollback

## 相關

- 設定參考：[../config-reference.md](../config-reference.md)
- 安全套件：[../security/README.md](../security/README.md)
