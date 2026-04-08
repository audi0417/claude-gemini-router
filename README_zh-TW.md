# Claude Gemini Router

> **[English](README.md)** | **[简体中文](README_zh-CN.md)**

一個會自進化的 Claude Code 插件，根據任務特徵智慧地將任務路由到 Google Gemini CLI，為大型上下文操作節省 Claude token。

**用越多次越聰明** — 系統會記住你的偏好，自動進化決策邏輯。

## 功能特色

- **智慧任務評分** — 自動評估任務更適合 Gemini（大檔案分析、摘要）還是 Claude（程式碼編輯、迭代工作流程）
- **偏好學習** — 透過經驗日誌記住你的選擇，自動路由熟悉的任務類型，無需再問
- **不滿意偵測** — 監測負面回饋，學會避免將類似任務路由到 Gemini
- **自進化** — 實現與 AutoSkill 研究相同的經驗驅動終身學習（ELL）原則

---

## 自進化學習迴圈

```
使用者請求 → 任務評分 → 經驗庫比對
                                ↓
                      ┌─ AUTO_APPROVE → 直接委派 Gemini
                      ├─ AVOID → Claude 自己處理
                      └─ 未知 → 詢問使用者
                                          ↓
                                ┌─ 記住並使用 → 寫入 AUTO_APPROVE
                                ├─ 只用一次 → 不記錄
                                └─ Claude 處理 → 不記錄
                                          ↓
                      使用者不滿意？ → 寫入 AVOID，未來自動迴避
```

### 為什麼說是「自進化」？

這個 Skill 實現了與 **AutoSkill**（Yang et al., 2026, arXiv:2603.01145）研究中提出的 **Experience-driven Lifelong Learning (ELL)** 相同的核心理念：

| AutoSkill 論文概念 | Gemini Router 實現 |
|-------------------|-------------------|
| **經驗偵測** — 監控互動中的穩定偏好 | 評分機制偵測任務特徵（檔案大小、讀/寫、複雜度） |
| **模式萃取** — 從回饋中提取可複用技能 | 使用者選「記住」時，萃取任務模式寫入 `AUTO_APPROVE` |
| **版本演化** — 技能隨回饋持續更新 | `AVOID` 可覆蓋 `AUTO_APPROVE`，決策邏輯持續修正 |
| **技能應用** — 未來任務自動套用已學習的技能 | 匹配到 `AUTO_APPROVE` 的任務直接委派，零確認 |

**關鍵共通點：**
- **人類監督** — 兩者都在記錄前徵求使用者同意，不會自作主張
- **雙向學習** — 不只記住成功（AUTO_APPROVE），也記住失敗（AVOID）
- **持久化記憶** — 經驗跨對話保存，不會用完即忘
- **漸進式進化** — 隨著使用次數增加，系統決策越來越精準

---

## 前置需求

- [Claude Code](https://claude.ai/code) 已安裝並完成認證
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) 已安裝（`npm install -g @google/gemini-cli`）

## 安裝

### 步驟一：新增 marketplace

在 Claude Code 中執行：

```
/plugin marketplace add audi0417/claude-gemini-router
```

### 步驟二：安裝插件

```
/plugin install gemini-router@claude-gemini-router
```

或使用互動式插件管理器：

```
/plugin
```

然後切換到 **Discover** 分頁，選擇 **gemini-router**。

### 替代方案：手動安裝

直接複製 skill 檔案：

```bash
git clone https://github.com/audi0417/claude-gemini-router.git
cp -r claude-gemini-router/plugins/gemini-router/skills/gemini ~/.claude/skills/gemini
```

## 運作方式

1. **評估** — 當你給 Claude 一個任務時，skill 會自動為其評分，判斷是否適合 Gemini
2. **決定** — 高分任務會提示你選擇：委派給 Gemini，或留給 Claude
3. **學習** — 你的選擇會被記錄，類似任務下次自動路由
4. **適應** — 如果 Gemini 的輸出不滿意，skill 會學會避免該類任務

### 評分標準

| 訊號 | 權重 |
|------|------|
| 大檔案（總計 >50KB） | +2 |
| 唯讀／分析任務 | +2 |
| 匹配 AUTO_APPROVE 模式 | +2 |
| 使用者明確要求使用 Gemini | +2 |
| 多檔案交叉比對 | +1 |
| 摘要／提取任務 | +1 |
| 節省 token 效益明顯 | +1 |

### 排除條件（一律由 Claude 處理）

- 匹配經驗庫中的 AVOID 模式
- 需要寫入或編輯檔案
- 需要多輪工具互動
- 簡短、簡單的任務

### 經驗庫（experience.md）

所有學習成果都存在 `experience.md` 中，格式透明、人類可讀：

```markdown
### [AUTO_APPROVE] 2026-04-08
**Pattern**: Full Go codebase read-only analysis
**Trigger keywords**: codebase, analysis, review
**Reason**: Large file volume, read-only, Gemini 1M context is a perfect fit

### [AVOID] 2026-04-08
**Pattern**: File writing, editing, or code generation
**What went wrong**: Gemini headless is single-shot, cannot iterate on edits
**Lesson**: Any task requiring Write/Edit tool use stays with Claude
```

---

## 使用範例

### 自動委派（已學習的模式）
```
你：幫我分析這 20 個檔案的架構
Claude：Routing to Gemini (auto-approved: large codebase analysis)
[Gemini Response] ...
```

### 首次遇到（詢問偏好）
```
你：幫我讀這個 200KB 的 log 找 error
Claude：這個任務看起來適合 Gemini（大量上下文）。要怎麼處理？
  1. 用 Gemini 並記住這類任務
  2. 用 Gemini，僅此一次
  3. Claude 自己處理
```

### 不滿意回饋
```
你：Gemini 的分析不太對，你重做一次
Claude：要不要把這類任務標記為不適合 Gemini？
  1. 好，記錄下來
  2. 不用，只是這次
```

---

## 授權

[MIT](LICENSE)
