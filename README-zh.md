# Switch-Omo-Config

用於切換 [oh-my-opencode](https://github.com/code-yeongyu/oh-my-opencode) 設定檔的互動式 CLI 工具。

![Version](https://img.shields.io/badge/version-3.2.0-blue) ![macOS](https://img.shields.io/badge/macOS-compatible-brightgreen) ![Shell](https://img.shields.io/badge/shell-bash-blue)

## 功能介紹

快速在不同的 `oh-my-opencode` 設定檔之間切換，無需手動複製檔案。當您擁有針對不同 AI 提供者（ChatGPT、Google、Copilot 等）的多個設定檔時非常有用。

## 先決條件

- macOS（使用 `md5` 進行檔案比對）
- 已安裝 [oh-my-opencode](https://github.com/code-yeongyu/oh-my-opencode)
- 至少有一個命名為 `oh-my-opencode-*.json` 的設定檔位於：
  - `~/.config/opencode/`（全域設定檔）
  - 專案中的 `.opencode/`（專案本地設定檔）

## 安裝

### 選項 1：直接下載

```bash
curl -o ~/.config/opencode/switch-omo-config.sh \
  https://raw.githubusercontent.com/AnPod/Switch-Omo-Config/main/switch-omo-config.sh
chmod +x ~/.config/opencode/switch-omo-config.sh
```

### 選項 2：手動安裝

```bash
# 將腳本複製到您的 opencode 設定目錄
cp switch-omo-config.sh ~/.config/opencode/
chmod +x ~/.config/opencode/switch-omo-config.sh
```

### 新增 Shell 別名（推薦）

```bash
# 適用於 zsh
echo 'alias omo-switch="~/.config/opencode/switch-omo-config.sh"' >> ~/.zshrc
source ~/.zshrc

# 適用於 bash
echo 'alias omo-switch="~/.config/opencode/switch-omo-config.sh"' >> ~/.bashrc
source ~/.bashrc
```

## 使用方式

```bash
# 直接執行
~/.config/opencode/switch-omo-config.sh

# 或使用別名
omo-switch
```

### 操作控制

| 按鍵       | 動作           |
| ---------- | -------------- |
| `↑` 或 `k` | 上移選擇       |
| `↓` 或 `j` | 下移選擇       |
| `Enter`    | 套用選擇的設定 |
| `q`        | 不變更並退出   |

### 範例輸出

```
Switch oh-my-opencode Configuration
Use arrow keys to navigate, Enter to select, q to quit

> oh-my-opencode-ChatGPT.json
  oh-my-opencode-baseline.json
  oh-my-opencode-copilot.json
  oh-my-opencode-google.json (active)
  oh-my-opencode-minimax.json
```

## 運作原理

1. 偵測您是否位於擁有 `.opencode/` 目錄的專案中。
2. 如果 `.opencode/` 不存在，詢問是否建立它以進行專案本地切換。
   - 您的回答（`y` 或 `n`）將被儲存至 `./.switch-omo-config.create-opencode`，以便在該目錄中不會再次詢問。
   - 如果您回答 `y`，腳本將建立 `./.opencode/` 並繼續在專案本地模式下運作。
3. （僅限專案本地，首次執行）詢問是否將全域設定檔 `~/.config/opencode/oh-my-opencode-*.json` 複製到 `./.opencode/`。
   - 您的回答（`y` 或 `n`）將被儲存至 `./.opencode/.switch-omo-config.copy-profiles`，以便在該專案中不會再次詢問。
4. 掃描所選目錄中的 `oh-my-opencode-*.json`，比對雜湊值以找出目前使用的設定檔，並將選擇的設定檔複製為同目錄下的 `oh-my-opencode.json`。

> **注意**：切換設定後，您必須退出並重新開啟 OpenCode 才能使變更生效。若要繼續上一次的對話，請使用 `/session` 並選擇最後一個工作階段。

## 重設專案提示

若要讓腳本在專案目錄中再次詢問，請刪除以下任一（或兩個）檔案：

- `./.switch-omo-config.create-opencode`（重新詢問是否建立 `./.opencode/`）
- `./.opencode/.switch-omo-config.copy-profiles`（重新詢問是否複製全域設定檔）

如果您想要完全重設該目錄的設定，也可以刪除 `./.opencode/`。

## 設定檔結構

### 全域模式（目前目錄無 `.opencode/`）

```
~/.config/opencode/
├── opencode.json              # 主要 OpenCode 設定
├── oh-my-opencode.json        # 目前使用的 oh-my-opencode 設定（由本工具管理）
├── oh-my-opencode-ChatGPT.json
├── oh-my-opencode-google.json
├── oh-my-opencode-copilot.json
└── oh-my-opencode-baseline.json
```

### 專案本地模式（當 `.opencode/` 存在時）

```
./.switch-omo-config.create-opencode  # 儲存建立提示的回答 (y/n)
./.opencode/
├── oh-my-opencode.json              # 目前使用的 oh-my-opencode 設定（由本工具管理）
├── oh-my-opencode-ChatGPT.json
├── oh-my-opencode-google.json
├── oh-my-opencode-copilot.json
├── oh-my-opencode-baseline.json
└── .switch-omo-config.copy-profiles # 儲存複製提示的回答 (y/n)
```

## 已知問題

### Anthropic "Invalid signature in thinking block" 錯誤

在同一工作階段切換回 Anthropic 模型後，您可能會遇到：

```
messages.1.content.0: Invalid 'signature' in 'thinking' block
```

**解決方案**：在您的 OpenCode 設定中停用 Anthropic 模型的 "thinking" 功能。

將以下內容新增至 `~/.config/opencode/opencode.json`（或專案的 `opencode.json`）並重新啟動 OpenCode：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "anthropic": {
      "models": {
        "claude-sonnet-4-5": {
          "options": {
            "thinking": { "type": "disabled" }
          }
        }
      }
    }
  }
}
```

## 更新日誌

- **v3.2.0** - 更新所有設定檔以使用改良的架構
- **v3.1.0** - 新增 sisyphus_agent、git_master 以及所有設定中的 categories
- **v3.0.0** - 大幅修訂 agent 設定架構
- **v2.0.0** - 架構重組與新的 agent 定義
- **v1.09** - 版本整合釋出
- **v1.08** - 新增完整的 Copilot 設定檔
- **v1.07** - 新增 `oh-my-opencode-full-copilot.json` 設定檔（所有 agent 皆使用 GitHub Copilot 模型）
- **v1.06** - 將 Google 模型升級為 `antigravity-*` 版本以提升效能
- **v1.05** - 將 `multimodal-looker` agent 從 `gemini-2.5-flash` 升級至 `gemini-3-flash`
- **v1.04** - 在 README 中新增版本徽章與更新日誌
- **v1.03** - 支援專案本地 `.opencode/` 目錄與建立提示
- **v1.02** - 標準化 agent 模型（`frontend-ui-ux-engineer`、`document-writer`、`multimodal-looker`）
- **v1.01** - 首次公開釋出

## 授權條款

MIT
