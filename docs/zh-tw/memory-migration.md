# 記憶體遷移指南

本指南說明 ZeroClaw 記憶體的結構以及如何在機器間遷移。

## ZeroClaw 記憶體是什麼

ZeroClaw 記憶體是 agent 的持久知識儲存。根據設定的 backend，儲存為：

- **Markdown backend**：`~/.zeroclaw/memory/` 目錄中的 `.md` 檔案
- **SQLite backend**：`~/.zeroclaw/memory.db`（或設定中指定的路徑）

## 找到您的記憶體儲存位置

查看 `~/.zeroclaw/config.toml` 中的 `[memory]` 區段：

```toml
[memory]
backend = "sqlite"      # sqlite | markdown | lucid | none
# 若有自訂路徑：
# path = "/custom/path/memory.db"
```

若無自訂路徑，預設位置為：

- SQLite：`~/.zeroclaw/memory.db`
- Markdown：`~/.zeroclaw/memory/`

## 備份記憶體

### SQLite Backend

```bash
cp ~/.zeroclaw/memory.db ~/.zeroclaw/memory.db.backup
```

### Markdown Backend

```bash
cp -r ~/.zeroclaw/memory/ ~/zeroclaw-memory-backup/
```

## 遷移到新機器

### SQLite Backend

```bash
# 在來源機器上
cp ~/.zeroclaw/memory.db /tmp/zeroclaw-memory.db

# 傳輸到新機器（例如使用 scp）
scp /tmp/zeroclaw-memory.db user@newmachine:~/.zeroclaw/memory.db
```

### Markdown Backend

```bash
# 在來源機器上打包
tar -czf /tmp/zeroclaw-memory.tar.gz -C ~/.zeroclaw memory/

# 傳輸到新機器
scp /tmp/zeroclaw-memory.tar.gz user@newmachine:/tmp/

# 在新機器上解壓
mkdir -p ~/.zeroclaw
tar -xzf /tmp/zeroclaw-memory.tar.gz -C ~/.zeroclaw/
```

## 從 OpenClaw 遷移

```bash
zeroclaw migrate openclaw --dry-run    # 先預覽
zeroclaw migrate openclaw              # 執行遷移
```

若 OpenClaw 資料在非預設位置：

```bash
zeroclaw migrate openclaw --source /path/to/openclaw/data
```

## 重要注意事項

### Embedding 向量

若在不同 embedding provider 之間遷移（例如從 OpenAI embeddings 換到 Ollama），儲存的向量將需要重新生成，因為向量空間不相容。

重新生成 embeddings：

```bash
# 遷移後，刪除現有向量並重建
zeroclaw memory rebuild-index
```

### 設定一致性

確保新機器的 `config.toml` 中 `[memory]` 區段設定與來源機器相同（相同的 backend 類型和路徑）。

## 相關文件

- [config-reference.md](config-reference.md) — 記憶體設定選項
- [commands-reference.md](commands-reference.md) — `migrate` 指令參考
