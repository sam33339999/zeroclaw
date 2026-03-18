# ZeroClaw 移除指南

本指南說明如何完全移除 ZeroClaw。

## 1. 停止服務

```bash
zeroclaw service stop
zeroclaw service uninstall
```

## 2. 移除二進位

若透過 `cargo install` 安裝：

```bash
cargo uninstall zeroclaw
```

或直接移除：

```bash
rm ~/.cargo/bin/zeroclaw
```

若透過 Homebrew 安裝：

```bash
brew uninstall zeroclaw
```

## 3. 移除設定和資料

```bash
rm -rf ~/.zeroclaw/
```

> ⚠️ 警告：此操作會永久刪除所有設定、記憶體和工作區資料，無法復原。

若只想移除設定但保留記憶體：

```bash
rm ~/.zeroclaw/config.toml
```

## 4. 可選：移除 Rust Toolchain

若不再需要 Rust：

```bash
rustup self uninstall
```

## 5. 清理頻道 Bot Token

以下 token 需手動在各平台撤銷：

- **Telegram**：透過 [@BotFather](https://t.me/BotFather) 刪除 bot
- **Discord**：在 [Discord Developer Portal](https://discord.com/developers/applications) 刪除應用程式
- **Slack**：在 Slack App 管理頁面移除 app
- **Matrix**：撤銷 access token（`/_matrix/client/v3/logout`）
- **Mattermost**：在 Mattermost 帳戶設定中撤銷 bot token

## 6. 驗證清理完成

```bash
which zeroclaw          # 應顯示找不到
ls ~/.zeroclaw/         # 應顯示不存在
```
