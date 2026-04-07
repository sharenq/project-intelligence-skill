# Risk Mode — 風險分析模式

## 目的

深入分析一次變更可能帶來的回歸風險、影響範圍，並提供具體的驗證建議。

Risk mode 聚焦在「這次變更會不會弄壞什麼」。

---

## 啟用方式

在呼叫 Core Analyzer 時指定 `mode: risk`。

---

## 額外分析要求

在 Core Analyzer 的基礎分析完成後，Risk mode 額外要求：

### 1. 深入影響範圍分析

- 這次變更直接影響哪些程式碼路徑？
- 有沒有間接影響？（例：改了共用函式，呼叫端可能受影響）
- 如果有 Project Profile，哪些已定義的 modules 在影響範圍內？
- 影響範圍的信心程度？（是否可能有你看不到的呼叫端？）

### 2. 回歸風險評估

- 這次變更有沒有改到已有功能的行為？
- 有沒有改變介面契約？（函式簽名、API 回應格式、事件結構等）
- 有沒有改變錯誤處理行為？
- 有沒有改變執行順序或時序？

### 3. 風險模式匹配

如果有 Project Profile 的 risk_patterns：
- 逐一檢查每個 risk_pattern 的 keywords 是否出現在變更中
- 匹配到的 pattern 要特別標記並說明

如果沒有 Profile：
- 檢查通用高風險模式：
  - 刪除操作
  - 權限/認證變更
  - 資料結構變更
  - 併發/非同步邏輯變更
  - 環境設定變更

### 4. 具體驗證建議

不只是「應該測試」，而是：
- 具體應該測試什麼情境？
- 應該用什麼方式驗證？（unit test、integration test、手動測試）
- 最優先的測試項目是什麼？
- 有沒有可以快速驗證的 smoke test？

---

## mode_specific 輸出結構

在 analysis-result 的 `mode_specific` 欄位中輸出：

```json
{
  "mode_specific": {
    "impact_analysis": {
      "direct_impact": ["直接受影響的檔案或模組"],
      "indirect_impact": ["可能間接受影響的區域"],
      "impact_confidence": "low | medium | high"
    },
    "regression_risks": [
      {
        "description": "可能的回歸風險",
        "likelihood": "low | medium | high",
        "impact": "low | medium | high",
        "affected_functionality": "受影響的功能描述"
      }
    ],
    "matched_risk_patterns": [
      {
        "pattern_name": "匹配到的風險模式名稱",
        "matched_keywords": ["匹配到的關鍵字"],
        "context": "在什麼脈絡下匹配到的"
      }
    ],
    "verification_plan": {
      "priority_tests": ["最優先的測試項目"],
      "smoke_tests": ["快速驗證方式"],
      "full_regression": ["完整回歸測試建議"]
    },
    "risk_summary": "一段話總結風險評估"
  }
}
```

---

## 風險分析原則

1. **寧可多報也不要漏報**：風險分析偏向保守，寧可標記一個最後證明沒問題的風險，也不要漏掉真正的問題
2. **區分可能性與影響**：likelihood 和 impact 分開評估，低可能性但高影響的風險仍需標記
3. **具體可驗證**：每個風險都要有對應的驗證方法，不能只是模糊的警告
4. **考慮看不到的部分**：diff 只顯示改了什麼，但受影響的可能包括沒改的程式碼。在 impact_confidence 中反映這種不確定性
5. **誠實標記信心**：如果你對影響範圍不確定，在 impact_confidence 中反映，不要假裝你知道所有呼叫端
