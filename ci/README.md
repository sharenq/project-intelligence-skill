# CI 整合指南

## 概覽

Project Intelligence 可以整合到 CI/CD 流程中，在 PR 建立或更新時自動執行分析。

---

## 整合架構

```
PR 建立/更新
    ↓
CI 擷取 diff
    ↓
載入 Project Profile
    ↓
執行 core-analyzer（review + risk mode）
    ↓
將結果發布為 PR comment
    ↓
（可選）執行 memory mode → knowledge-merger
```

---

## 前置條件

1. **AI 執行能力**：CI 環境需要能執行 AI prompt。可選方案：
   - Claude Code CLI（推薦）
   - Anthropic API + 自訂 script
   - 其他相容的 AI CLI 工具

2. **Project Profile**：專案根目錄或指定路徑下需有對應的 profile JSON

3. **Git 歷史**：checkout 時需要 `fetch-depth: 0` 以取得完整 diff

---

## GitHub Actions

參考 `github-actions.yml` 範例。主要步驟：

1. Checkout（含完整歷史）
2. 產生 PR diff
3. 執行 review mode 分析
4. 執行 risk mode 分析
5. 將結果發布為 PR comment

---

## 結果輸出格式

CI 分析結果遵循 `schemas/analysis-result.schema.json`，可以：

- 直接以 JSON 存為 CI artifact
- 解析後格式化為人類可讀的 PR comment
- 傳入 knowledge-merger 進行知識沉澱

### PR Comment 建議格式

```markdown
## Project Intelligence Analysis

**信心程度**：medium
**意圖**：新增匯出功能端點

### 風險
- [high] 大量資料匯出可能導致記憶體問題
- [medium] 端點未實作分頁

### 建議測試
- 測試大量資料情境下的匯出行為
- 測試無資料時的回應格式

### 待確認
- 匯出的資料量預期有多大？
- 是否需要非同步處理？
```

---

## 觸發策略建議

| 策略 | 適用場景 | 說明 |
|------|---------|------|
| 每次 PR 更新 | 活躍開發期 | 每次 push 都跑分析 |
| 僅首次開 PR | 降低成本 | 只在 PR opened 時跑 |
| 手動觸發 | 大型專案 | 透過 PR label 或 comment 觸發 |
| 僅特定路徑 | Monorepo | 只在關鍵路徑變更時觸發 |

### Monorepo 路徑過濾範例

```yaml
on:
  pull_request:
    paths:
      - 'packages/core/**'
      - 'packages/api/**'
      - '!**/*.md'
      - '!**/*.test.*'
```

---

## 成本控制

AI 分析有 token 成本。建議：

1. **限制 diff 大小**：超過一定行數的 diff 截斷或摘要
2. **快取 Profile**：Profile 不需要每次重新載入
3. **選擇性執行**：不是每次 push 都需要跑完整分析
4. **mode 選擇**：日常 PR 只跑 review，重要 PR 再加 risk

---

## 其他 CI 平台

`github-actions.yml` 是 GitHub Actions 範例。核心邏輯可移植到：

- GitLab CI（`.gitlab-ci.yml`）
- CircleCI（`.circleci/config.yml`）
- Jenkins（`Jenkinsfile`）

關鍵步驟相同：擷取 diff → 載入 profile → 執行分析 → 發布結果。
