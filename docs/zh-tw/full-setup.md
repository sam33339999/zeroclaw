# ZeroClaw 完整安裝指南

本指南涵蓋從安裝到驗證的完整設定流程。

## 簡介

ZeroClaw 是一個 Rust 優先的自主 agent runtime，針對效能、效率、穩定性、可擴充性和安全性進行最佳化。

## 安裝

請參閱 [one-click-bootstrap.md](one-click-bootstrap.md) 取得詳細的安裝說明。

快速安裝：

```bash
git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw
./install.sh
```

或透過 Homebrew：

```bash
brew install zeroclaw
```

## Provider 設定

ZeroClaw 支援多種 AI provider。完整清單請參閱 [providers-reference.md](providers-reference.md)。

### 雲端 Provider

| Provider ID | 環境變數 | 說明 |
|---|---|---|
| `openrouter` | `OPENROUTER_API_KEY` | 多模型 gateway |
| `anthropic` | `ANTHROPIC_API_KEY` | Claude 系列 |
| `openai` | `OPENAI_API_KEY` | GPT 系列 |
| `gemini` | `GEMINI_API_KEY` | Google Gemini |
| `groq` | `GROQ_API_KEY` | 高速推理 |
| `deepseek` | `DEEPSEEK_API_KEY` | DeepSeek 系列 |
| `xai` | `XAI_API_KEY` | Grok 系列 |
| `mistral` | `MISTRAL_API_KEY` | Mistral 系列 |
| `zai` | `ZAI_API_KEY` | Z.AI / GLM 系列 |
| `glm` | `GLM_API_KEY` | Zhipu AI GLM |
| `moonshot` | `MOONSHOT_API_KEY` | Kimi 系列 |
| `qwen` | `DASHSCOPE_API_KEY` | Qwen 系列 |
| `together` | `TOGETHER_API_KEY` | Together AI |
| `fireworks` | `FIREWORKS_API_KEY` | Fireworks AI |
| `perplexity` | `PERPLEXITY_API_KEY` | Perplexity |
| `cohere` | `COHERE_API_KEY` | Cohere |
| `nvidia` | `NVIDIA_API_KEY` | NVIDIA NIM |
| `bedrock` | `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY` | AWS Bedrock |

### 本機 Provider

| Provider ID | 說明 |
|---|---|
| `ollama` | 本機 Ollama 伺服器 |
| `lmstudio` | LM Studio 本機伺服器 |
| `custom:<url>` | 自訂 OpenAI 相容 endpoint |
| `anthropic-custom:<url>` | 自訂 Anthropic 相容 endpoint |

### 快速設定

```bash
zeroclaw onboard --api-key "your-key" --provider openrouter
```

## 通訊頻道設定

ZeroClaw 支援多種通訊頻道。完整設定請參閱 [channels-reference.md](channels-reference.md)。

| 頻道 | 說明 | 設定指南 |
|---|---|---|
| Telegram | 透過 Telegram bot 通訊 | [channels-reference.md](channels-reference.md#41-telegram) |
| Discord | 透過 Discord bot 通訊 | [channels-reference.md](channels-reference.md#42-discord) |
| Slack | 透過 Slack app 通訊 | [channels-reference.md](channels-reference.md#43-slack) |
| Mattermost | Self-hosted 通訊 | [mattermost-setup.md](mattermost-setup.md) |
| Matrix | 含 E2EE 支援 | [matrix-e2ee-guide.md](matrix-e2ee-guide.md) |
| Signal | 透過 signal-cli bridge | [channels-reference.md](channels-reference.md#46-signal) |
| WhatsApp | Cloud API 或 Web 模式 | [channels-reference.md](channels-reference.md#47-whatsapp) |
| Webhook | HTTP webhook endpoint | [channels-reference.md](channels-reference.md#48-webhook) |
| Email | IMAP/SMTP | [channels-reference.md](channels-reference.md#49-email) |
| IRC | IRC 網路 | [channels-reference.md](channels-reference.md#410-irc) |
| Lark/Feishu | 飛書整合 | [channels-reference.md](channels-reference.md#411-lark--feishu) |
| DingTalk | 釘釘整合 | [channels-reference.md](channels-reference.md#412-dingtalk) |
| QQ | QQ bot | [channels-reference.md](channels-reference.md#413-qq) |
| iMessage | 本機 iMessage | [channels-reference.md](channels-reference.md#414-imessage) |

## 基本驗證

```bash
zeroclaw status
zeroclaw doctor
zeroclaw channel doctor
```

## 網路訪問設定

ZeroClaw 預設拒絕所有 HTTP 請求（`[http_request]` 預設停用且 `allowed_domains` 為空）。若需要 agent 能呼叫外部 API，請明確啟用並設定允許的網域：

```toml
[http_request]
enabled = true
allowed_domains = ["api.example.com", "api.another.com"]
```

若您的網路環境需要 Proxy：

```toml
[proxy]
enabled = true
scope = "zeroclaw"
http_proxy = "http://proxy.corp.example.com:7890"
https_proxy = "http://proxy.corp.example.com:7890"
no_proxy = ["localhost", "127.0.0.1"]
```

詳細說明請參閱 [network-access.md](network-access.md)。

## 後續步驟

- [network-access.md](network-access.md) — 網路訪問與 Proxy 設定指南
- [commands-reference.md](commands-reference.md) — 完整指令參考
- [config-reference.md](config-reference.md) — 設定選項
- [operations-runbook.md](operations-runbook.md) — 運維手冊
- [troubleshooting.md](troubleshooting.md) — 疑難排解
