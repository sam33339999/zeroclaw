# ZeroClaw 設定參考

常用設定項目和預設值。

最後驗證：**2026-02-20**。

## 設定檔位置

`~/.zeroclaw/config.toml`

## 核心設定

```toml
# Provider 設定
api_key = "your-api-key"
default_provider = "openrouter"
default_model = "anthropic/claude-sonnet-4-6"
default_temperature = 0.7

# Agent 設定
[agent]
compact_context = false
autonomy_level = "supervised"   # readonly | supervised | full
max_actions_per_hour = 20

# Gateway 設定
[gateway]
host = "127.0.0.1"
port = 3000
allow_public_bind = false

# Memory 設定
[memory]
backend = "sqlite"              # sqlite | markdown | lucid | none
embedding_model = "none"

# 可靠性設定
[reliability]
max_retries = 3
retry_delay_ms = 1000
fallback_providers = []
```

## Autonomy 設定

```toml
[autonomy]
level = "supervised"
allowed_commands = ["git", "ls", "cat", "grep", "find"]
forbidden_paths = ["/etc", "/root", "~/.ssh"]
require_approval_for_medium_risk = true
block_high_risk_commands = true
max_actions_per_hour = 20
```

## 頻道設定

詳細頻道設定請參閱 [channels-reference.md](channels-reference.md)。

## 相關文件

- [commands-reference.md](commands-reference.md)
- [channels-reference.md](channels-reference.md)
- [providers-reference.md](providers-reference.md)
