# Project Intelligence Skill

一個通用型 AI 專案理解 skill。讓 AI 能結構化地分析任何軟體專案的變更意圖、影響範圍與風險。

**不綁定任何特定產業、商業流程或資料模型。** 專案差異透過 Project Profile 調整，核心分析引擎保持通用。

---

## 快速開始

### 1. 為你的專案建立 Profile

把 `profiles/` 中的 sample 複製一份，根據你的專案修改：

```bash
cp profiles/backend-service.sample.json profiles/my-project.json
```

編輯 `my-project.json`，填入你的專案資訊。最少只需要：

```json
{
  "project_name": "my-project",
  "modules": [
    {
      "name": "core",
      "keywords": ["核心邏輯的關鍵字"],
      "important_paths": ["src/core/"]
    }
  ]
}
```

或者讓 AI 幫你生成：

> 請使用 profile-generator，根據這個 repo 的結構生成一份 Project Profile。

### 2. 分析程式碼變更

將 Core Analyzer prompt 與你選擇的 mode 提供給 AI，搭配你的 Project Profile：

**Review 模式（預設）**— 專案感知型程式碼審查：
> 請使用 project-intelligence 的 review mode，分析這次的 git diff。
> Profile: [貼上或指向你的 profile]

**Risk 模式** — 回歸風險與影響範圍分析：
> 請使用 project-intelligence 的 risk mode，分析這次變更的風險。

**Memory 模式** — 萃取可沉澱的知識：
> 請使用 project-intelligence 的 memory mode，從這次分析中提取值得記住的知識。

### 3. 取得結構化輸出

所有 mode 都會輸出符合 `schemas/analysis-result.schema.json` 的 JSON，包含：

| 欄位 | 說明 |
|------|------|
| `summary` | 一段式摘要 |
| `intent` | 變更意圖 |
| `confidence` | 整體信心程度 |
| `affected_modules` | 受影響的模組 |
| `behavioral_observations` | 觀察到的行為模式 |
| `risks` | 識別到的風險 |
| `suggested_tests` | 建議的測試方向 |
| `assumptions` | 分析中做出的假設 |
| `open_questions` | 無法回答的問題 |
| `mode_specific` | 各 mode 的專屬輸出 |

---

## 檔案結構

```
project-intelligence-skill/
├── README.md                          ← 你正在讀的文件
├── skill.yaml                         ← Skill 定義與設定
├── prompts/system/
│   ├── core-analyzer.md               ← 核心分析引擎（所有 mode 共用）
│   └── profile-generator.md           ← Profile 自動生成器
├── schemas/
│   ├── project-profile.schema.json    ← Profile 結構定義
│   └── analysis-result.schema.json    ← 分析結果結構定義
├── modes/
│   ├── memory.md                      ← 知識沉澱模式
│   ├── review.md                      ← 程式碼審查模式
│   └── risk.md                        ← 風險分析模式
└── profiles/
    ├── backend-service.sample.json    ← 後端服務範例
    ├── frontend-app.sample.json       ← 前端應用範例
    └── monorepo.sample.json           ← Monorepo 範例
```

---

## 設計原則

### Domain-Agnostic（去領域化）
核心 prompt 與 schema 不包含任何產業、商業流程或特定資料模型假設。如果你的專案有特定的領域術語或流程，透過 Profile 的 `domain_terms` 和 `risk_patterns` 提供。

### Honest Reasoning（誠實推理）
資訊不足時，分析結果必須包含：
- `assumptions`：做出了哪些假設
- `open_questions`：無法回答的問題
- `confidence`：整體信心程度

AI 不會假裝知道它不知道的事。

### Profile-Driven（Profile 驅動）
同一個分析引擎，透過不同的 Profile 適應不同專案。換專案只需換 Profile，不需改 prompt。

### Structured Output（結構化輸出）
所有分析結果都符合 `analysis-result.schema.json`，可以被程式解析、存檔、比對。

---

## Project Profile 指南

Profile 是分析引擎理解你專案的關鍵。但它不需要一步到位。

### 最小可用 Profile

```json
{
  "project_name": "my-project"
}
```

只有名稱也能跑分析，只是精確度較低。

### 建議逐步補充的欄位

| 優先順序 | 欄位 | 為什麼重要 |
|----------|------|-----------|
| 1 | `modules` | 讓分析能正確映射影響範圍 |
| 2 | `tech_stack` | 讓風險模式更精確 |
| 3 | `risk_patterns` | 讓分析知道你的專案特別在意什麼 |
| 4 | `domain_terms` | 讓分析理解專案特有術語 |
| 5 | `analysis_preferences` | 微調分析行為 |

### 用 Profile Generator 自動生成

提供 README、目錄樹、或依賴檔給 profile-generator，它會生成初始 Profile 並標記哪些部分需要人工確認。

---

## Modes 說明

### review（預設）
**用途**：專案感知型程式碼審查

不只看語法和風格，還會考慮變更在專案脈絡中的意義。輸出包含 review items（含嚴重程度與具體建議）和整體評估。

### risk
**用途**：回歸風險與影響範圍分析

深入分析變更可能帶來的回歸風險，提供具體的驗證計畫。適合在合併重要變更前使用。

### memory
**用途**：知識沉澱

從分析結果中萃取值得長期保留的知識片段（行為規則、架構決策、已知陷阱）。適合在完成重要功能或修復後使用。

---

## 範例：分析一次 git diff

**輸入**：
```
請使用 project-intelligence 的 review mode 分析以下 diff：

Profile: profiles/backend-service.sample.json

[貼上 git diff 內容]
```

**輸出**（簡化）：
```json
{
  "summary": "在 API 路由中新增了一個端點，用於處理資料匯出請求",
  "intent": "新增功能",
  "confidence": "medium",
  "affected_modules": ["api", "services"],
  "behavioral_observations": [
    {
      "title": "匯出端點未實作分頁",
      "confidence": "high",
      "evidence": "handler 中直接查詢全部資料，無 limit/offset 參數"
    }
  ],
  "risks": [
    {
      "title": "大量資料匯出可能導致記憶體問題",
      "severity": "high",
      "affected_area": "api"
    }
  ],
  "suggested_tests": [
    "測試大量資料情境下的匯出行為",
    "測試無資料時的回應格式"
  ],
  "assumptions": [
    "假設此端點會被直接呼叫（非背景任務）"
  ],
  "open_questions": [
    "匯出的資料量預期有多大？",
    "是否需要非同步處理？"
  ]
}
```

---

## 延後項目（不在 v0.1 範圍）

以下功能已規劃但不在第一版：

- doc mode（文件生成）
- onboarding mode（新人引導）
- refactor mode（重構分析）
- 自動知識合併
- CI 整合
- 編排層（orchestration）

---

## 授權

MIT
