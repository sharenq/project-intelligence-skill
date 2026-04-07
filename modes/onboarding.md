# Onboarding Mode — 新人引導模式

## 目的

幫助剛加入專案的工程師快速建立對專案的基本理解。產出一份結構化的「專案入門指南」，讓新人知道從哪裡開始、什麼是重要的、什麼要小心。

---

## 啟用方式

在呼叫 Core Analyzer 時指定 `mode: onboarding`。

---

## 建議輸入

Onboarding mode 最適合接收以下輸入（越多越好）：

1. **repo 目錄樹** — 用於理解專案結構
2. **README** — 用於理解專案目的
3. **Project Profile** — 用於理解模組與術語
4. **主要進入點檔案**（如 main.ts, app.py, index.js）— 用於理解啟動流程
5. **設定檔**（如 package.json, docker-compose.yml）— 用於理解開發環境

---

## 額外分析要求

在 Core Analyzer 的基礎分析完成後，Onboarding mode 額外要求：

### 1. 專案全貌

- 這個專案是做什麼的？（一段話）
- 主要技術棧是什麼？
- 程式碼大致分成哪幾個區域？
- 哪些區域是核心、哪些是輔助？

### 2. 新人導覽路線

建議新人按什麼順序認識專案：
- 先看哪些檔案？
- 先理解哪些概念？
- 哪些模組可以晚一點再看？

### 3. 重要約定與慣例

- 專案中有沒有明顯的命名慣例？
- 程式碼組織有什麼規律？
- 有沒有不明顯但重要的約定？（如果能從程式碼中觀察到的話）

### 4. 注意事項

- 哪些區域比較複雜或容易出錯？
- 有沒有已知的技術債或歷史包袱？（僅限於從程式碼中能觀察到的）
- 新人最容易踩到什麼坑？

### 5. 開發環境

- 如何啟動專案？
- 需要什麼前置條件？
- 如何跑測試？
- 有沒有需要設定的環境變數？

---

## mode_specific 輸出結構

在 analysis-result 的 `mode_specific` 欄位中輸出：

```json
{
  "mode_specific": {
    "project_overview": "一段話描述專案全貌",
    "learning_path": [
      {
        "order": 1,
        "target": "應該先看的檔案或模組",
        "reason": "為什麼要先看這個",
        "estimated_complexity": "low | medium | high"
      }
    ],
    "conventions": [
      {
        "convention": "觀察到的約定",
        "evidence": "從哪裡觀察到的",
        "confidence": "low | medium | high"
      }
    ],
    "pitfalls": [
      {
        "description": "新人容易踩到的坑",
        "area": "相關區域",
        "suggestion": "如何避免"
      }
    ],
    "dev_setup": {
      "prerequisites": ["前置條件"],
      "start_commands": ["啟動指令"],
      "test_commands": ["測試指令"],
      "env_vars": ["需要設定的環境變數（不含實際值）"]
    },
    "onboarding_completeness": "low | medium | high",
    "gaps": ["這份指南缺少什麼、建議補充什麼"]
  }
}
```

---

## 引導原則

1. **新人視角**：假設讀者聰明但對此專案一無所知
2. **路徑優先**：給出明確的閱讀順序，不要丟出一堆清單讓新人自己判斷
3. **誠實標記盲區**：你看不到的部分（如部署流程、團隊約定）在 gaps 中列出
4. **不要假裝全知**：onboarding 指南中推斷的部分必須標記 confidence
5. **實用性測試**：問自己「如果我是新人，看到這份指南能不能開始工作？」
