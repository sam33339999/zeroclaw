# 網路訪問設定 — ZeroClaw

本指南說明如何在 ZeroClaw 中設定網路訪問，包括啟用 HTTP 請求 tool、設定允許的網域，以及透過 Proxy 路由流量。

最後更新：**2026-03-18**。

---

## 為什麼網路訪問預設受限？

ZeroClaw 採用**預設拒絕**的安全模型：

- `[http_request]` tool 預設停用（`enabled = false`）
- `allowed_domains` 預設為空列表（`[]`），即使啟用後，所有 HTTP 請求也會被拒絕
- 即使設定 `"*"` 允許所有公開網域，本地/私有 IP（如 `127.0.0.1`、`192.168.x.x`、`10.x.x.x`）仍會被封鎖

這意味著若未明確設定 `[http_request]`，agent 無法進行任何 HTTP 請求。

---

## 快速路徑：根據需求選擇設定方式

| 我想要… | 跳至 |
|---|---|
| 讓 agent 能呼叫特定 API | [第 1 節：啟用 http_request 並設定允許網域](#1-啟用-http_request-並設定允許網域) |
| 允許 agent 訪問任何公開網站 | [第 2 節：允許所有公開網域](#2-允許所有公開網域) |
| 透過 Proxy 路由 ZeroClaw 流量 | [第 3 節：Proxy 設定](#3-proxy-設定) |
| 了解哪些網路請求會被封鎖 | [第 4 節：安全限制說明](#4-安全限制說明) |
| 排查「無法連線」問題 | [第 5 節：疑難排解](#5-疑難排解) |

---

## 1. 啟用 `http_request` 並設定允許網域

編輯 `~/.zeroclaw/config.toml`：

```toml
[http_request]
enabled = true
allowed_domains = ["api.openai.com", "api.anthropic.com"]
max_response_size = 1000000   # 1 MB，預設值
timeout_secs = 30              # 秒，預設值
```

### 網域匹配規則

- **精確匹配**：`"api.example.com"` 只允許 `api.example.com`
- **子網域匹配**：`"example.com"` 允許 `example.com` 及其所有子網域（`api.example.com`、`www.example.com` 等）
- **萬用字元**：`"*"` 允許所有公開網域（但私有 IP 仍被封鎖）

### 常用設定範例

#### 只允許特定 API

```toml
[http_request]
enabled = true
allowed_domains = [
  "api.github.com",
  "api.stripe.com",
  "hooks.slack.com",
]
```

#### 允許某個網站的所有子網域

```toml
[http_request]
enabled = true
allowed_domains = ["example.com"]
# 允許：api.example.com、www.example.com、cdn.example.com 等
```

#### 允許所有公開網域（開發/測試用）

```toml
[http_request]
enabled = true
allowed_domains = ["*"]
# 注意：localhost、127.0.0.1、私有 IP 仍被封鎖
```

---

## 2. 允許所有公開網域

若需要 agent 能訪問任意公開網站（例如開發環境或測試），使用萬用字元：

```toml
[http_request]
enabled = true
allowed_domains = ["*"]
```

> ⚠️ **安全注意**：在生產環境中建議使用精確的網域列表，而非 `"*"`。
> 即使設定 `"*"`，以下目標仍會被自動封鎖：
> - `localhost`、`127.0.0.1`（loopback）
> - 私有 IP 範圍：`10.x.x.x`、`172.16-31.x.x`、`192.168.x.x`
> - IPv6 本地位址

---

## 3. Proxy 設定

若您的網路環境需要透過 Proxy 才能訪問外部網路，可設定 `[proxy]`。

### 基本 Proxy 設定（`config.toml`）

```toml
[proxy]
enabled = true
scope = "zeroclaw"           # zeroclaw | services | environment
http_proxy = "http://proxy.corp.example.com:7890"
https_proxy = "http://proxy.corp.example.com:7890"
no_proxy = ["localhost", "127.0.0.1", ".internal.corp"]
```

### Proxy 範圍（`scope`）

| 範圍 | 作用對象 | 是否匯出環境變數 | 典型使用場景 |
|---|---|---|---|
| `zeroclaw` | ZeroClaw 內部 HTTP 客戶端 | 否 | 一般 runtime Proxy，無 process 層面副作用 |
| `services` | 只有明確列出的服務 key/selector | 否 | 針對特定 provider/tool/channel 的細粒度路由 |
| `environment` | Runtime + process 環境變數 | 是 | 需要 `HTTP_PROXY`/`HTTPS_PROXY`/`ALL_PROXY` 的整合 |

### Proxy 範圍範例

#### 只代理 ZeroClaw 內部流量（推薦）

```toml
[proxy]
enabled = true
scope = "zeroclaw"
http_proxy = "http://127.0.0.1:7890"
https_proxy = "http://127.0.0.1:7890"
no_proxy = ["localhost", "127.0.0.1"]
```

#### 只代理特定服務

```toml
[proxy]
enabled = true
scope = "services"
services = ["provider.openai", "tool.http_request", "channel.telegram"]
all_proxy = "socks5h://127.0.0.1:1080"
no_proxy = ["localhost", "127.0.0.1", ".internal"]
```

#### 匯出 Proxy 環境變數給整個 process

```toml
[proxy]
enabled = true
scope = "environment"
http_proxy = "http://127.0.0.1:7890"
https_proxy = "http://127.0.0.1:7890"
no_proxy = "localhost,127.0.0.1,.internal"
```

### 支援的 Proxy 協定

- `http://` — HTTP Proxy
- `https://` — HTTPS Proxy
- `socks5://` — SOCKS5 Proxy
- `socks5h://` — SOCKS5 Proxy（由 Proxy 解析 DNS）

### 透過 agent tool 在 runtime 管理 Proxy

ZeroClaw 提供 `proxy_config` tool，可在 runtime 動態切換 Proxy 設定，而無需重啟：

```json
{"action":"get"}
```

```json
{"action":"set","enabled":true,"scope":"zeroclaw","http_proxy":"http://127.0.0.1:7890","https_proxy":"http://127.0.0.1:7890"}
```

```json
{"action":"disable"}
```

詳細操作手冊請參閱 [proxy-agent-playbook.md](proxy-agent-playbook.md)。

---

## 4. 安全限制說明

### 永遠被封鎖的目標

無論 `allowed_domains` 如何設定，以下目標永遠被封鎖：

- **Loopback**：`127.0.0.1`、`::1`、`localhost`
- **私有 IPv4 範圍**：
  - `10.0.0.0/8`
  - `172.16.0.0/12`
  - `192.168.0.0/16`
  - `169.254.0.0/16`（link-local）
- **保留的 IPv6 位址**：本地連結（`fe80::/10`）、唯一本地（`fc00::/7`）等

### `allowed_domains` 為空時

若 `allowed_domains = []`（預設），即使 `enabled = true`，所有 HTTP 請求都會被拒絕，錯誤訊息類似：

```
http_request: domain 'api.example.com' is not in the allowed domains list
```

### 瀏覽器 tool 也有獨立的網域控制

`[browser]` 和 `[http_request]` 是獨立的設定區塊：

```toml
[browser]
enabled = true
allowed_domains = ["example.com"]   # 瀏覽器只能開啟 example.com

[http_request]
enabled = true
allowed_domains = ["api.example.com"]   # http_request 只能呼叫 api.example.com
```

---

## 5. 疑難排解

### 問題：agent 說無法發送 HTTP 請求

**症狀**：

```
Error: http_request tool is not enabled
```

**解決方案**：在 `config.toml` 中設定：

```toml
[http_request]
enabled = true
allowed_domains = ["your-api-domain.com"]
```

然後重啟 runtime：

```bash
zeroclaw service restart
# 或
zeroclaw daemon
```

---

### 問題：網域不在允許列表中

**症狀**：

```
http_request: domain 'api.example.com' is not in the allowed domains list
```

**解決方案**：將網域加入 `allowed_domains`：

```toml
[http_request]
enabled = true
allowed_domains = ["api.example.com"]
```

---

### 問題：需要連線到公司內網但被封鎖

**症狀**：需要訪問 `192.168.x.x` 或 `10.x.x.x` 的私有 API，但請求被拒絕。

**說明**：私有/本地 IP 是硬性封鎖的安全邊界，**無法透過設定繞過**。

**建議方案**：
- 使用公開可訪問的 API endpoint（反向代理到公開 URL）
- 若必須訪問內網資源，考慮使用 Tailscale、ngrok 等穿透工具建立可公開訪問的 URL

---

### 問題：透過 Proxy 後仍無法連線

**排查步驟**：

1. 確認 Proxy 設定正確：
   ```bash
   # 在 agent chat 中執行
   # 呼叫 proxy_config tool
   {"action":"get"}
   ```

2. 確認 `http_request` 也已啟用並設定了 `allowed_domains`：
   ```toml
   [http_request]
   enabled = true
   allowed_domains = ["*"]

   [proxy]
   enabled = true
   scope = "zeroclaw"
   http_proxy = "http://127.0.0.1:7890"
   ```

3. 確認 `no_proxy` 沒有意外排除目標網域。

4. 重啟 runtime 讓設定生效：
   ```bash
   zeroclaw service restart
   ```

---

### 問題：Proxy URL scheme 不正確

**症狀**：

```
Error: invalid proxy URL scheme
```

**解決方案**：使用支援的協定（`http`、`https`、`socks5`、`socks5h`）：

```toml
# 正確
http_proxy = "http://127.0.0.1:7890"
all_proxy = "socks5h://127.0.0.1:1080"

# 錯誤（不支援）
http_proxy = "tcp://127.0.0.1:7890"
```

---

## 6. 完整設定範例

以下是同時設定 `http_request` 和 `proxy` 的完整範例：

```toml
# ~/.zeroclaw/config.toml

default_provider = "openrouter"
default_model = "anthropic/claude-sonnet-4-6"
api_key = "sk-or-..."

[http_request]
enabled = true
allowed_domains = [
  "api.openai.com",
  "api.anthropic.com",
  "api.github.com",
  "raw.githubusercontent.com",
]
max_response_size = 5000000   # 5 MB
timeout_secs = 60

[proxy]
enabled = true
scope = "zeroclaw"
http_proxy = "http://127.0.0.1:7890"
https_proxy = "http://127.0.0.1:7890"
no_proxy = ["localhost", "127.0.0.1", "::1"]
```

---

## 相關文件

- [config-reference.md](config-reference.md) — 完整設定參考，包含所有設定鍵值
- [proxy-agent-playbook.md](proxy-agent-playbook.md) — Proxy 操作手冊（runtime 動態管理）
- [network-deployment.md](network-deployment.md) — 網路部署指南（Raspberry Pi、LAN、Tunnel）
- [troubleshooting.md](troubleshooting.md) — 常見問題排查
- [operations-runbook.md](operations-runbook.md) — 日常維運手冊
