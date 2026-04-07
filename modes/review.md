# Review Mode — 專案感知型程式碼審查模式

## 目的

進行一次有專案上下文意識的程式碼審查。不是語法層面的 linting，而是從「這段變更對專案的影響」角度進行審查。

---

## 啟用方式

在呼叫 Core Analyzer 時指定 `mode: review`。

---

## 額外分析要求

在 Core Analyzer 的基礎分析完成後，Review mode 額外要求：

### 1. 識別可疑變更

- 變更意圖不清楚的部分
- 看起來是暫時性 workaround 的程式碼
- 缺乏錯誤處理的新邏輯
- 硬編碼的值（magic numbers、hardcoded strings）
- 改了介面但可能沒更新所有呼叫端的情況

### 2. 評估結構品質

- 這次變更是否讓模組職責更清楚還是更模糊？
- 有沒有引入不必要的耦合？
- 函式 / 檔案大小是否合理？
- 命名是否清楚表達意圖？

### 3. 檢查與專案慣例的一致性

- 如果有 Project Profile，檢查變更是否符合已知的專案模式
- 如果有 domain_terms，檢查命名是否一致
- 如果有 risk_patterns 匹配，特別標記

### 4. 給出具體建議

- 每個問題點都要有具體的改善建議
- 區分嚴重程度：critical / high / medium / low
- 區分類型：bug / risk / maintainability / clarity

---

## mode_specific 輸出結構

在 analysis-result 的 `mode_specific` 欄位中輸出：

```json
{
  "mode_specific": {
    "review_items": [
      {
        "severity": "critical | high | medium | low",
        "category": "bug | risk | maintainability | clarity | consistency",
        "location": "檔案路徑或程式碼位置",
        "issue": "問題描述",
        "suggestion": "具體改善建議",
        "rationale": "為什麼這是問題"
      }
    ],
    "overall_assessment": "pass | pass_with_warnings | needs_changes",
    "review_summary": "一段話總結審查結果"
  }
}
```

---

## 審查原則

1. **專案感知優先**：不要只做表面的 code style review，要考慮這段變更在專案脈絡中的意義
2. **具體可行動**：每個 review item 都要有具體的 suggestion，不要只說「這裡有問題」
3. **區分輕重**：不要把所有問題都標 critical，讓工程師能優先處理重要的
4. **尊重意圖**：先理解變更者想做什麼，再評估做法是否合適
5. **承認不確定性**：如果你不確定某段程式碼是否有問題，把它放在 open_questions 而不是 review_items
