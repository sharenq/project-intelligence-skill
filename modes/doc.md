# Doc Mode — 文件生成模式

## 目的

根據分析結果，生成對團隊有實際幫助的技術文件草稿。不是 API doc 自動生成工具，而是從「理解程式碼後寫出人類需要的說明」角度產出文件。

---

## 啟用方式

在呼叫 Core Analyzer 時指定 `mode: doc`。

---

## 額外分析要求

在 Core Analyzer 的基礎分析完成後，Doc mode 額外要求：

### 1. 判斷適合的文件類型

根據輸入內容，判斷最適合產出哪種文件：

| 輸入類型 | 建議文件類型 |
|----------|-------------|
| 單一模組的程式碼 | 模組說明文件 |
| 跨模組的 diff | 變更說明 / changelog 條目 |
| 整個 repo 結構 | 架構概覽 |
| 特定功能的程式碼 | 功能說明文件 |
| 設定檔 | 設定指南 |

### 2. 撰寫文件草稿

文件草稿必須：
- 以使用者（工程師）角度撰寫，不是 AI 角度
- 聚焦「為什麼」和「怎麼用」，而非逐行解釋程式碼
- 包含必要的前置條件或相依資訊
- 標明哪些部分是推斷的、需要人工確認

### 3. 標記文件品質

- 這份文件的完整度如何？
- 哪些段落需要專案成員補充？
- 有沒有遺漏的重要主題？

---

## mode_specific 輸出結構

在 analysis-result 的 `mode_specific` 欄位中輸出：

```json
{
  "mode_specific": {
    "doc_type": "module_overview | changelog_entry | architecture_overview | feature_guide | config_guide",
    "title": "文件標題",
    "content_markdown": "完整的 Markdown 文件內容",
    "completeness": "low | medium | high",
    "sections_needing_review": [
      {
        "section": "段落標題或描述",
        "reason": "為什麼需要人工審查"
      }
    ],
    "suggested_follow_up_docs": ["建議後續可產出的文件"]
  }
}
```

---

## 文件撰寫原則

1. **讀者優先**：假設讀者是熟悉程式設計但不熟悉此專案的工程師
2. **實用大於完整**：一份能用的 70% 完成度草稿 > 一份看起來完整但空泛的文件
3. **標記推斷**：任何基於推測寫出的內容，用 `<!-- TODO: 請確認 -->` 標記
4. **不要複製程式碼**：文件解釋概念和流程，不是把程式碼再貼一次
5. **保持結構一致**：使用標準的 Markdown 結構（標題、清單、程式碼區塊）
