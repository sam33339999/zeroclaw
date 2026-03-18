# ZeroClaw 設定參考

常用設定項目和預設值。

最後驗證：**2026-02-19**。

啟動時的設定檔搜尋順序：

1. 環境變數 `ZEROCLAW_WORKSPACE`（若已設定）
2. 標記檔 `~/.zeroclaw/active_workspace.toml`（若存在）
3. 預設 `~/.zeroclaw/config.toml`

ZeroClaw 啟動時會以 `INFO` 等級記錄已解析的設定檔路徑：

- `Config loaded`，包含欄位：`path`、`workspace`、`source`、`initialized`

匯出 schema 指令：

- `zeroclaw config schema`（將 JSON Schema draft 2020-12 輸出至 stdout）

## 核心鍵值

| 鍵值 | 預設值 | 說明 |
|---|---|---|
| `default_provider` | `openrouter` | Provider ID 或別名 |
| `default_model` | `anthropic/claude-sonnet-4-6` | 透過所選 provider 路由的模型 |
| `default_temperature` | `0.7` | 模型溫度 |

## `[observability]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `backend` | `none` | 監測後端：`none`、`noop`、`log`、`prometheus`、`otel`、`opentelemetry` 或 `otlp` |
| `otel_endpoint` | `http://localhost:4318` | backend 為 `otel` 時的 OTLP HTTP endpoint |
| `otel_service_name` | `zeroclaw` | 傳送至 OTLP collector 的服務名稱 |

注意：

- `backend = "otel"` 使用具有阻塞式匯出用戶端的 OTLP HTTP 匯出，讓 span 和 metric 可從 Tokio context 外部安全傳送。
- `opentelemetry` 和 `otlp` 別名均指向相同的 OTel backend。

範例：

```toml
[observability]
backend = "otel"
otel_endpoint = "http://localhost:4318"
otel_service_name = "zeroclaw"
```

## 透過環境變數覆寫 Provider

也可透過環境變數選擇 Provider。優先順序：

1. `ZEROCLAW_PROVIDER`（明確覆寫，有值時永遠優先）
2. `PROVIDER`（舊版後備，僅在 config 中的 provider 未設定或仍為 `openrouter` 時套用）
3. `config.toml` 中的 `default_provider`

容器使用者注意事項：

- 若 `config.toml` 設定了自訂 provider 如 `custom:https://.../v1`，Docker/容器預設的 `PROVIDER=openrouter` 環境變數不會覆蓋它。
- 若想讓環境變數覆寫已設定的 provider，請使用 `ZEROCLAW_PROVIDER`。

## `[agent]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `compact_context` | `false` | 啟用時：bootstrap_max_chars=6000，rag_chunk_limit=2。適用於 13B 以下模型 |
| `max_tool_iterations` | `10` | CLI、gateway 和 channels 每則訊息的最大 tool-call 迴圈次數 |
| `max_history_messages` | `50` | 每個 session 保留的最大歷史訊息數量 |
| `parallel_tools` | `false` | 在單一輪次中啟用平行 tool 執行 |
| `tool_dispatcher` | `auto` | Tool dispatch 策略 |
| `tool_call_dedup_exempt` | `[]` | 在同一輪次中免除重複呼叫檢查的 tool 名稱 |

注意：

- 設定 `max_tool_iterations = 0` 將使用安全預設值 `10`。
- 若 channel 訊息超過此值，runtime 會回傳：`Agent exceeded maximum tool iterations (<value>)`。
- 在 CLI、gateway 和 channel 的 tool 迴圈中，獨立的 tool 呼叫預設會在不需要審核時並行執行；結果順序保持穩定。
- `parallel_tools` 適用於 `Agent::turn()` API，不影響 CLI、gateway 或 channel 的 runtime 迴圈。
- `tool_call_dedup_exempt` 接受精確的 tool 名稱陣列。列表中的 tool 在同一輪次中可以用相同參數多次呼叫。範例：`tool_call_dedup_exempt = ["browser"]`。

## `[agents.<name>]`

子代理（sub-agent）設定。`[agents]` 下的每個鍵定義一個有名稱的子代理，主代理可以委派給它。

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `provider` | _必填_ | Provider 名稱（例如 `"ollama"`、`"openrouter"`、`"anthropic"`） |
| `model` | _必填_ | 子代理使用的模型名稱 |
| `system_prompt` | 未設定 | 子代理的自訂 system prompt（選填） |
| `api_key` | 未設定 | 自訂 API key（`secrets.encrypt = true` 時加密） |
| `temperature` | 未設定 | 子代理的自訂溫度 |
| `max_depth` | `3` | 巢狀委派的最大遞迴深度 |
| `agentic` | `false` | 為子代理啟用多輪 tool-call 迴圈模式 |
| `allowed_tools` | `[]` | agentic 模式下允許的 tool 列表 |
| `max_iterations` | `10` | agentic 模式下的最大 tool-call 輪次 |

注意：

- `agentic = false` 保留單輪 prompt→response 委派行為。
- `agentic = true` 至少需要 `allowed_tools` 中有一個匹配項。
- `delegate` tool 會從子代理的 allowlist 中排除，以避免委派迴圈。

```toml
[agents.researcher]
provider = "openrouter"
model = "anthropic/claude-sonnet-4-6"
system_prompt = "You are a research assistant."
max_depth = 2
agentic = true
allowed_tools = ["web_search", "http_request", "file_read"]
max_iterations = 8

[agents.coder]
provider = "ollama"
model = "qwen2.5-coder:32b"
temperature = 0.2
```

## `[runtime]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `reasoning_enabled` | 未設定（`None`） | 對支援的 provider 全域覆寫 reasoning/thinking |

注意：

- `reasoning_enabled = false` 明確停用支援的 provider 端 reasoning（目前為 `ollama`，透過 `think: false` 欄位）。
- `reasoning_enabled = true` 要求明確 reasoning（`ollama` 上的 `think: true`）。
- 未設定則保留 provider/model 的預設行為。

## `[skills]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `open_skills_enabled` | `false` | 允許載入/同步社群 `open-skills` 儲存庫 |
| `open_skills_dir` | 未設定 | `open-skills` 的本地路徑（啟用時預設為 `$HOME/open-skills`） |

注意：

- 安全預設：ZeroClaw **不會** clone 或同步 `open-skills`，除非 `open_skills_enabled = true`。
- 透過環境變數覆寫：
  - `ZEROCLAW_OPEN_SKILLS_ENABLED` 接受 `1/0`、`true/false`、`yes/no`、`on/off`。
  - `ZEROCLAW_OPEN_SKILLS_DIR` 有值時覆寫儲存庫路徑。
- 優先順序：`ZEROCLAW_OPEN_SKILLS_ENABLED` → `config.toml` 中的 `skills.open_skills_enabled` → 預設 `false`。

## `[composio]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `enabled` | `false` | 啟用 Composio 管理的 OAuth tool |
| `api_key` | 未設定 | `composio` tool 的 Composio API key |
| `entity_id` | `default` | 呼叫 connect/execute 時傳送的預設 `user_id` |

注意：

- 向後相容：舊版 `enable = true` 可接受為 `enabled = true` 的別名。
- 若 `enabled = false` 或缺少 `api_key`，`composio` tool 不會被註冊。
- ZeroClaw 需要使用 `toolkit_versions=latest` 的 Composio v3 tools，並以 `version="latest"` 執行以避免舊的預設 tool 版本。
- 一般流程：呼叫 `connect`、在瀏覽器完成 OAuth，然後對所需操作執行 `execute`。
- 若 Composio 回傳缺少 connected-account 的錯誤，呼叫 `list_accounts`（選填 `app`）並將回傳的 `connected_account_id` 傳給 `execute`。

## `[cost]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `enabled` | `false` | 啟用費用追蹤 |
| `daily_limit_usd` | `10.00` | 每日消費上限（USD） |
| `monthly_limit_usd` | `100.00` | 每月消費上限（USD） |
| `warn_at_percent` | `80` | 消費達此百分比時發出警告 |
| `allow_override` | `false` | 允許在使用 `--override` 旗標時超出預算 |

注意：

- `enabled = true` 時，runtime 追蹤每個請求的估計費用並套用每日/每月上限。
- 達到 `warn_at_percent` 閾值時發出警告，但請求仍繼續。
- 達到上限時請求被拒絕，除非 `allow_override = true` 且傳入 `--override` 旗標。

## `[identity]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `format` | `openclaw` | 身分格式：`"openclaw"`（預設）或 `"aieos"` |
| `aieos_path` | 未設定 | AIEOS JSON 檔路徑（相對於 workspace） |
| `aieos_inline` | 未設定 | 內聯 AIEOS JSON（檔路徑的替代方案） |

注意：

- 使用 `format = "aieos"` 搭配 `aieos_path` 或 `aieos_inline` 以載入 AIEOS / OpenClaw 身分文件。
- `aieos_path` 和 `aieos_inline` 只應設定其中一個；`aieos_path` 優先。

## `[multimodal]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `max_images` | `4` | 每個請求的最大圖片 marker 數量 |
| `max_image_size_mb` | `5` | base64 編碼前的圖片大小限制 |
| `allow_remote_fetch` | `false` | 允許從 marker 中的 `http(s)` URL 取得圖片 |

注意：

- Runtime 接受訊息中使用語法 ``[IMAGE:<source>]`` 的圖片 marker。
- 支援的來源：
  - 本地檔路徑（例如 ``[IMAGE:/tmp/screenshot.png]``）
  - Data URI（例如 ``[IMAGE:data:image/png;base64,...]``）
  - 遠端 URL 僅在 `allow_remote_fetch = true` 時允許
- 允許的 MIME 類型：`image/png`、`image/jpeg`、`image/webp`、`image/gif`、`image/bmp`。
- 當使用中的 provider 不支援 vision 時，請求會以結構化的 capability 錯誤（`capability=vision`）失敗，而不是靜默忽略圖片。

## `[browser]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `enabled` | `false` | 啟用 `browser_open` tool（在系統預設瀏覽器中開啟 URL，不抓取資料） |
| `allowed_domains` | `[]` | `browser_open` 的允許網域（精確或子網域匹配） |
| `session_name` | 未設定 | 瀏覽器 session 名稱（用於 agent-browser 自動化） |
| `backend` | `agent_browser` | 自動化後端：`"agent_browser"`、`"rust_native"`、`"computer_use"` 或 `"auto"` |
| `native_headless` | `true` | rust-native 後端的無頭模式 |
| `native_webdriver_url` | `http://127.0.0.1:9515` | rust-native 後端的 WebDriver endpoint URL |
| `native_chrome_path` | 未設定 | rust-native 後端的選填 Chrome/Chromium 路徑 |

### `[browser.computer_use]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `endpoint` | `http://127.0.0.1:8787/v1/actions` | computer-use 動作的 sidecar endpoint（OS 層級的滑鼠/鍵盤/截圖） |
| `api_key` | 未設定 | computer-use sidecar 的選填 Bearer token（儲存時加密） |
| `timeout_ms` | `15000` | 每個動作的逾時時間（毫秒） |
| `allow_remote_endpoint` | `false` | 允許 sidecar 的遠端/公開 endpoint |
| `window_allowlist` | `[]` | 允許 sidecar 互動的視窗標題/程序 allowlist |
| `max_coordinate_x` | 未設定 | 座標型動作的 X 軸限制（選填） |
| `max_coordinate_y` | 未設定 | 座標型動作的 Y 軸限制（選填） |

注意：

- `backend = "computer_use"` 時，agent 將瀏覽器動作委派給 `computer_use.endpoint` 處的 sidecar。
- `allow_remote_endpoint = false`（預設）拒絕所有非 loopback endpoint 以防止意外暴露。
- 使用 `window_allowlist` 限制 sidecar 可互動的 OS 視窗。

## `[http_request]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `enabled` | `false` | 啟用 `http_request` tool 進行 API 互動 |
| `allowed_domains` | `[]` | 允許的網域（精確或子網域匹配，或 `"*"` 允許所有公開網域） |
| `max_response_size` | `1000000` | 最大回應大小（位元組，預設：1 MB） |
| `timeout_secs` | `30` | 請求逾時（秒） |

注意：

- **預設拒絕全部**：若 `allowed_domains` 為空，所有 HTTP 請求均被拒絕。這是網路訪問受到安全限制的最常見原因。
- 使用精確網域或子網域匹配（例如 `"api.example.com"`、`"example.com"`），或使用 `"*"` 允許任何公開網域。
- 即使設定 `"*"`，本地/私有目標仍會被封鎖。

若要啟用網路訪問，請參閱 [network-access.md](network-access.md)。

## `[gateway]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `host` | `127.0.0.1` | 綁定位址 |
| `port` | `3000` | Gateway 監聽埠 |
| `require_pairing` | `true` | Bearer 驗證前需要配對 |
| `allow_public_bind` | `false` | 防止意外公開暴露 |

## `[autonomy]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `level` | `supervised` | `read_only`、`supervised` 或 `full` |
| `workspace_only` | `true` | 限制寫入/指令在 workspace 範圍內 |
| `allowed_commands` | _執行 shell 必填_ | 允許的指令列表 |
| `forbidden_paths` | `[]` | 禁止的路徑列表 |
| `max_actions_per_hour` | `100` | 每小時動作預算 |
| `max_cost_per_day_cents` | `1000` | 每日消費限制（分） |
| `require_approval_for_medium_risk` | `true` | 中等風險指令需要審核 |
| `block_high_risk_commands` | `true` | 硬性封鎖高風險指令 |
| `auto_approve` | `[]` | 永遠自動核准的 tool 操作 |
| `always_ask` | `[]` | 永遠需要審核的 tool 操作 |

注意：

- `level = "full"` 跳過 shell 執行的中等風險審核，但仍套用已設定的 guardrail。
- Shell 分隔符/運算子解析能感知引號。引號參數中的字元如 `;` 被視為字面量，而非指令分隔符。
- 未引號的 shell 串接運算子仍受 policy 檢查（`;`、`|`、`&&`、`||`、背景執行和重新導向）。

## `[memory]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `backend` | `sqlite` | `sqlite`、`lucid`、`markdown`、`none` |
| `auto_save` | `true` | 僅儲存使用者輸入（assistant 輸出被排除） |
| `embedding_provider` | `none` | `none`、`openai` 或自訂 endpoint |
| `embedding_model` | `text-embedding-3-small` | 嵌入模型 ID，或 `hint:<name>` 路由 |
| `embedding_dimensions` | `1536` | 所選嵌入模型的預期向量大小 |
| `vector_weight` | `0.7` | 混合排名中的向量權重 |
| `keyword_weight` | `0.3` | 混合排名中的關鍵字權重 |

注意：

- 記憶體上下文注入會忽略舊版 `assistant_resp*` 自動儲存鍵，以防止模型生成的摘要被視為事實。

## `[[model_routes]]` 和 `[[embedding_routes]]`

路由提示讓整合名稱在模型 ID 變化時保持穩定。

### `[[model_routes]]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `hint` | _必填_ | 任務提示名稱（例如 `"reasoning"`、`"fast"`、`"code"`、`"summarize"`） |
| `provider` | _必填_ | 目標 provider（必須符合已知的 provider 名稱） |
| `model` | _必填_ | 使用該 provider 的模型 |
| `api_key` | 未設定 | 此路由 provider 的選填 API key 覆寫 |

### `[[embedding_routes]]`

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `hint` | _必填_ | 路由提示名稱（例如 `"semantic"`、`"archive"`、`"faq"`） |
| `provider` | _必填_ | 嵌入 provider（`"none"`、`"openai"` 或 `"custom:<url>"`） |
| `model` | _必填_ | 使用該 provider 的嵌入模型 |
| `dimensions` | 未設定 | 此路由的選填嵌入維度覆寫 |
| `api_key` | 未設定 | 此路由 provider 的選填 API key 覆寫 |

```toml
[memory]
embedding_model = "hint:semantic"

[[model_routes]]
hint = "reasoning"
provider = "openrouter"
model = "provider/model-id"

[[embedding_routes]]
hint = "semantic"
provider = "openai"
model = "text-embedding-3-small"
dimensions = 1536
```

升級策略：

1. 保持提示穩定（`hint:reasoning`、`hint:semantic`）。
2. 只更新路由項目中的 `model = "...新版本..."`。
3. 重啟/部署前以 `zeroclaw doctor` 驗證。

## `[query_classification]`

根據內容模式自動將訊息路由至 `[[model_routes]]` 提示。

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `enabled` | `false` | 啟用自動查詢分類 |
| `rules` | `[]` | 分類規則（按優先順序評估） |

`rules` 中每個規則：

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `hint` | _必填_ | 必須符合 `[[model_routes]]` 中的提示值 |
| `keywords` | `[]` | 不分大小寫的子字串匹配 |
| `patterns` | `[]` | 區分大小寫的精確字串匹配（用於 code fence、如 `"fn "` 的關鍵字） |
| `min_length` | 未設定 | 僅在訊息長度 ≥ N 字元時匹配 |
| `max_length` | 未設定 | 僅在訊息長度 ≤ N 字元時匹配 |
| `priority` | `0` | 優先順序較高的規則先被檢查 |

```toml
[query_classification]
enabled = true

[[query_classification.rules]]
hint = "reasoning"
keywords = ["explain", "analyze", "why"]
min_length = 200
priority = 10

[[query_classification.rules]]
hint = "fast"
keywords = ["hi", "hello", "thanks"]
max_length = 50
priority = 5
```

## `[channels_config]`

頂層 channel 設定位於 `channels_config` 下。

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `message_timeout_secs` | `300` | channel 訊息處理的基本逾時時間（秒）；runtime 根據 tool-loop 深度自動調整（最多 4x） |

範例：

- `[channels_config.telegram]`
- `[channels_config.discord]`
- `[channels_config.whatsapp]`
- `[channels_config.email]`

注意：

- 預設 `300s` 針對本地 LLM（Ollama）最佳化，比 cloud API 慢。
- Runtime 逾時預算為 `message_timeout_secs * scale`，其中 `scale = min(max_tool_iterations, 4)` 且最小值為 `1`。
- 此調整可避免在第一輪 LLM 慢速/重試但後續 tool-loop 輪次仍需完成時發生錯誤逾時。
- 若使用 cloud API（OpenAI、Anthropic 等），可降至 `60` 或更低。
- 低於 `30` 的值會被限制為 `30` 以避免持續逾時。
- 逾時發生時，使用者會收到：`⚠️ Request timed out while waiting for the model. Please try again.`
- 僅 Telegram 的中斷行為由 `channels_config.telegram.interrupt_on_new_message` 控制（預設 `false`）。啟用時，來自同一 chat 中同一發送者的新訊息會取消正在處理的請求並保留被中斷的使用者上下文。
- 當 `zeroclaw channel start` 執行時，對 `default_provider`、`default_model`、`default_temperature`、`api_key`、`api_url` 和 `reliability.*` 的變更可以從 `config.toml` 在下一則訊息時熱套用。

詳細的 channel 矩陣和 allowlist 行為請參閱 [channels-reference.md](channels-reference.md)。

### `[channels_config.whatsapp]`

WhatsApp 在同一個設定表下支援兩種後端。

Cloud API 模式（Meta webhook）：

| 鍵值 | 必填 | 用途 |
|---|---|---|
| `access_token` | 是 | Meta Cloud API Bearer token |
| `phone_number_id` | 是 | Meta 電話號碼 ID |
| `verify_token` | 是 | Webhook 驗證 token |
| `app_secret` | 選填 | 啟用 webhook 簽章驗證（`X-Hub-Signature-256`） |
| `allowed_numbers` | 建議 | 允許的來電號碼（`[]` = 拒絕全部，`"*"` = 允許全部） |

WhatsApp Web 模式（原生客戶端）：

| 鍵值 | 必填 | 用途 |
|---|---|---|
| `session_path` | 是 | 永久儲存的 SQLite session 路徑 |
| `pair_phone` | 選填 | pair-code 流程的電話號碼（僅數字） |
| `pair_code` | 選填 | 自訂配對碼（若未指定則自動生成） |
| `allowed_numbers` | 建議 | 允許的來電號碼（`[]` = 拒絕全部，`"*"` = 允許全部） |

注意：

- WhatsApp Web 需要 build flag `whatsapp-web`。
- 若同時設定了 Cloud 和 Web，為向後相容 Cloud 優先。

## `[hardware]`

實體硬體存取設定（STM32、probe、serial）。

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `enabled` | `false` | 啟用硬體存取 |
| `transport` | `none` | 傳輸模式：`"none"`、`"native"`、`"serial"` 或 `"probe"` |
| `serial_port` | 未設定 | Serial 埠路徑（例如 `"/dev/ttyACM0"`） |
| `baud_rate` | `115200` | Serial 鮑率 |
| `probe_target` | 未設定 | Probe 的目標晶片（例如 `"STM32F401RE"`） |
| `workspace_datasheets` | `false` | 啟用 workspace datasheet RAG（索引 PDF 原理圖供 AI 查詢引腳） |

注意：

- 使用 `transport = "serial"` 搭配 `serial_port` 進行 USB-serial 連線。
- 使用 `transport = "probe"` 搭配 `probe_target` 透過 debug-probe（例如 ST-Link）燒錄。
- 協定詳情請參閱 [hardware-peripherals-design.md](hardware-peripherals-design.md)。

## `[peripherals]`

啟用時，周邊板成為 agent tool。

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `enabled` | `false` | 啟用周邊支援（板成為 agent tool） |
| `boards` | `[]` | 板設定列表 |
| `datasheet_dir` | 未設定 | Datasheet 文件路徑（相對於 workspace）用於 RAG |

`boards` 中每個項目：

| 鍵值 | 預設值 | 用途 |
|---|---|---|
| `board` | _必填_ | 板類型：`"nucleo-f401re"`、`"rpi-gpio"`、`"esp32"` 等 |
| `transport` | `serial` | 傳輸類型：`"serial"`、`"native"`、`"websocket"` |
| `path` | 未設定 | Serial 路徑：`"/dev/ttyACM0"`、`"/dev/ttyUSB0"` |
| `baud` | `115200` | Serial 鮑率 |

```toml
[peripherals]
enabled = true
datasheet_dir = "docs/datasheets"

[[peripherals.boards]]
board = "nucleo-f401re"
transport = "serial"
path = "/dev/ttyACM0"
baud = 115200

[[peripherals.boards]]
board = "rpi-gpio"
transport = "native"
```

注意：

- 將以板命名的 `.md`/`.txt` datasheet 檔（例如 `nucleo-f401re.md`、`rpi-gpio.md`）放在 `datasheet_dir` 中以支援 RAG。
- 板協定和韌體說明請參閱 [hardware-peripherals-design.md](hardware-peripherals-design.md)。

## 安全相關預設值

- Channel allowlist 預設拒絕全部（`[]` 表示拒絕全部）
- Gateway 預設需要配對
- 預設封鎖公開綁定

## 驗證指令

修改設定後：

```bash
zeroclaw status
zeroclaw doctor
zeroclaw channel doctor
zeroclaw service restart
```

## 相關文件

- [network-access.md](network-access.md)
- [channels-reference.md](channels-reference.md)
- [providers-reference.md](providers-reference.md)
- [operations-runbook.md](operations-runbook.md)
- [troubleshooting.md](troubleshooting.md)
