# Memory Mode — 知識沉澱模式

## 目的

將分析結果轉化為可長期保留、供未來 AI 或工程師參考的結構化知識片段。

Memory mode 不是寫文件，而是萃取「值得記住的事」。

---

## 啟用方式

在呼叫 Core Analyzer 時指定 `mode: memory`。

---

## 額外分析要求

在 Core Analyzer 的基礎分析完成後，Memory mode 額外要求：

### 1. 萃取可重用知識

從本次分析中識別：
- **行為規則**：這段程式碼建立了什麼持久性的行為約束？（例：「所有 API 回應必須經過此 middleware」）
- **架構決策**：是否能看出某個設計決策？（例：「選擇 event-driven 而非 polling」）
- **已知陷阱**：是否有容易出錯的地方值得記錄？

### 2. 判斷沉澱價值

不是所有分析結果都值得沉澱。請判斷：
- 這個觀察是一次性的還是長期有效的？
- 這個規則是局部的還是跨模組的？
- 未來的 AI 或工程師是否會因為不知道這件事而犯錯？

只沉澱高價值、長期有效的知識。

### 3. 格式化為記憶片段

每個值得沉澱的知識點，格式化為一個記憶片段。

---

## mode_specific 輸出結構

在 analysis-result 的 `mode_specific` 欄位中輸出：

```json
{
  "mode_specific": {
    "memory_entries": [
      {
        "type": "behavioral_rule | architecture_decision | known_pitfall | pattern",
        "title": "簡短標題",
        "content": "詳細描述",
        "scope": "此知識適用的範圍（模組名、全域等）",
        "confidence": "low | medium | high",
        "evidence": "支持此記憶的依據",
        "durability": "long_term | medium_term | short_term"
      }
    ],
    "knowledge_summary": "本次分析新增了哪些知識（一句話）"
  }
}
```

---

## 沉澱原則

1. **寧缺勿濫**：5 條高價值記憶 > 20 條低訊號記憶
2. **標記信心**：不確定的知識仍可沉澱，但必須標記 confidence
3. **標記有效期**：
   - `long_term`：架構決策、核心行為規則
   - `medium_term`：目前的實作慣例（可能隨重構改變）
   - `short_term`：特定版本的注意事項
4. **避免重複**：如果某個知識已經很明顯（如「這是一個 Express app」），不需要沉澱
5. **聚焦隱性知識**：顯而易見的東西不需要記，需要記的是看不出來但很重要的事
