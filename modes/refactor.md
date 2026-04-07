# Refactor Mode — 重構分析模式

## 目的

識別程式碼中的重構機會，分析責任混雜、重複邏輯、過度耦合等結構問題，並提供具體的重構建議與風險評估。

Refactor mode 不是自動重構工具，而是「幫你看出哪裡值得重構、怎麼重構比較安全」。

---

## 啟用方式

在呼叫 Core Analyzer 時指定 `mode: refactor`。

---

## 額外分析要求

在 Core Analyzer 的基礎分析完成後，Refactor mode 額外要求：

### 1. 結構問題識別

掃描輸入中的結構問題：

| 問題類型 | 訊號 |
|----------|------|
| 職責混雜 | 一個檔案 / 函式同時處理多個不相關的事 |
| 過度耦合 | 模組 A 必須知道模組 B 的內部實作細節 |
| 重複邏輯 | 相似的程式碼片段出現在多處 |
| 過大單元 | 函式超過 50 行、檔案超過 800 行 |
| 深層嵌套 | 超過 4 層的條件 / 迴圈嵌套 |
| 命名不一致 | 同一概念在不同地方用不同名稱 |
| 抽象洩漏 | 高層模組直接操作低層實作細節 |
| 抽象層級不一致 | 同一函式或模組中混合了不同抽象層級的操作（如高層流程控制與低層字串處理並存） |

### 2. 重構機會排序

對識別出的問題，根據以下維度排序：

- **影響範圍**：重構此處能改善多大範圍的程式碼品質？
- **風險等級**：重構此處的回歸風險有多高？
- **難度**：重構需要多少工作量？
- **價值**：重構後帶來的可維護性提升有多大？

### 3. 具體重構建議

每個重構機會都要提供：
- 問題是什麼
- 建議的重構策略，對應經典重構模式（Extract Method、Extract Class、Move Module、Introduce Interface、Introduce Parameter Object、Replace Conditional with Polymorphism、Pull Up / Push Down 等）
- 重構步驟的概要
- 重構前應確保的測試覆蓋
- 可能的風險

### 4. 依賴關係分析

- 被重構的程式碼有哪些呼叫端？
- 重構會改變哪些介面？
- 哪些模組需要連帶調整？

---

## mode_specific 輸出結構

在 analysis-result 的 `mode_specific` 欄位中輸出：

```json
{
  "mode_specific": {
    "refactor_opportunities": [
      {
        "title": "重構機會標題",
        "problem_type": "mixed_responsibility | tight_coupling | duplication | oversized_unit | deep_nesting | naming_inconsistency | leaky_abstraction | abstraction_inconsistency",
        "location": "檔案路徑或程式碼位置",
        "description": "問題描述",
        "strategy": "建議的重構策略",
        "steps": ["重構步驟概要"],
        "impact_scope": "low | medium | high",
        "risk_level": "low | medium | high",
        "effort": "low | medium | high",
        "value": "low | medium | high",
        "prerequisites": ["重構前需要確保的事項（如測試覆蓋）"],
        "affected_dependents": ["可能需要連帶調整的模組或檔案"]
      }
    ],
    "priority_order": ["按建議優先順序排列的重構機會 title"],
    "refactor_summary": "一段話總結重構建議",
    "safe_to_start": true,
    "blockers": ["如果 safe_to_start 為 false，列出阻礙因素"]
  }
}
```

---

## 重構分析原則

1. **觀察優先於假設**：只根據看到的程式碼提出重構建議，不要假設「應該」長什麼樣
2. **安全優先於完美**：建議的重構路徑必須考慮回歸風險，先穩後美
3. **漸進式**：大型重構拆成可獨立驗證的小步驟
4. **測試前置**：每個重構建議都要說明需要什麼測試覆蓋才能安全進行
5. **不過度重構**：如果程式碼運作良好且容易理解，即使不「優雅」也不需要重構
6. **量化判斷**：用 impact/risk/effort/value 四個維度幫助決策，而不是靠直覺
