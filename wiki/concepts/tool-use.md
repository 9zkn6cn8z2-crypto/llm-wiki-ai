---
title: Tool Use (工具調用 / 函數調用)
aliases: [Function Calling, 工具使用, AI 工具調用]
type: concept
tags: [ai, agent, tool-use, function-calling, mcp]
created: 2026-11-23
updated: 2026-11-23
status: stable
sources:
  - "[[raw/papers/mcp-landscape-security-2025.md]]"
---

# Tool Use (工具調用 / 函數調用)

## 一句話白話直覺 (Intuition)
如果說原生的大語言模型（[[大語言模型-llm]]）是一個**「被關在玻璃房裡的超級天才腦袋」**（只能動嘴聊天、背出以前學過的事，但無法走出房間）；那麼 **Tool Use（工具調用）** 就是**給了這個天才一雙機械手和工具箱**。當他遇到算不出的複雜乘法時能拿出計算機，想知道今天天氣時能打開手機查即時氣象，甚至能親自操作終端機去修改你的電腦檔案。

---

## 它解決了過去什麼痛點？

### 傳統純文本 LLM 的致命缺陷：
1. **知識截止與即時性缺失（Knowledge Cutoff）**：模型權重一旦訓練完成，就無法知道今天發生的新聞或最新的股價。
2. **算術與邏輯死角（Bad at Exact Math）**：LLM 本質上是「文字機率預測器」，算微積分或 12 位數乘法容易胡言亂語（幻覺）。
3. **只能被動回答，無法主動行動（Action Gap）**：傳統對話機器人即使知道你想要寄信或訂機票，它也無法幫你按下「送出」按鈕。
4. **與 [[檢索增強生成-rag]] 的關鍵差異**：
   - 傳統 RAG 只能**被動閱讀**外部文件並摘錄；
   - Tool Use 讓模型能夠**主動改變外部世界**（例如寫入資料庫、發送 Slack 訊息、執行 Bash 腳本）。

---

## 核心機制與工作原理 (Mechanism)

很多人以為模型調用工具是「AI 自己直接跑去點滑鼠」，其實背後的技術機制非常巧妙且結構化：

```text
[使用者提問: "幫我查台北現在氣溫，並算如果加 5 度是多少？"]
                             │
                             ▼
┌────────────────────────────────────────────────────────┐
│ 1. 意圖識別與結構化輸出 (Function Calling)             │
│    模型判斷自己無法得知即時氣溫，決定呼叫工具：        │
│    輸出 JSON 格式：                                     │
│    { "tool": "get_weather", "location": "Taipei" }     │
└────────────────────────────┬───────────────────────────┘
                             │
                             ▼
┌────────────────────────────────────────────────────────┐
│ 2. 外部環境執行 (Host / Environment Execution)         │
│    程式（Python/系統）接收到 JSON，實際去連氣象局 API   │
│    取得回傳值：{ "temp": 24 }                          │
└────────────────────────────┬───────────────────────────┘
                             │
                             ▼
┌────────────────────────────────────────────────────────┐
│ 3. 觀察結果回饋與最終回答 (Observation & Answer)       │
│    將結果回餵給 LLM，模型進行第二輪計算推理：           │
│    "台北目前氣溫為 24°C，若加 5 度則是 29°C。"         │
└────────────────────────────────────────────────────────┘
```

### 工具調用的演進三部曲：
1. **手動 API 串接時代 (Manual Wiring, 2023 前)**：開發者必須為每個 API 寫死正則表達式或繁瑣的介面轉換。
2. **結構化 Function Calling 時代 (OpenAI, 2023)**：模型官方支援直接輸出標準 JSON Schema，工具調用穩定度大幅飛躍。
3. **標準化協議時代 ([[Model Context Protocol|MCP]], 2024~至今)**：擺脫每家模型與工具各自為政的混亂局面，採用通用的隨插即用介面。

---

## 形式化定義與數學基礎 (Mathematical Foundations)

在底層推論中，工具調用本質上是**在語法限制（Constrained Decoding）下的條件機率分佈預測與環境轉移**：

### 1. 工具選擇的條件機率分佈 (Tool Selection Probability)
設可用工具集合為 $\mathcal{T} = \{t_1, t_2, \dots, t_K\}$，每個工具具備描述文檔 $d_i$ 與參數格式 $\text{Schema}_i$。在給定使用者上下文 $c = (x_1, \dots, x_t)$ 下，模型決策是否調用工具 $t_i$ 或直接回覆自然語言（$t_0 = \text{Text}$）：
$$P_\theta(t_i \mid c, \mathcal{T}) = \frac{\exp\left( \frac{\text{sim}(f_\theta(c), g(d_i))}{\tau} \right)}{\sum_{j=0}^K \exp\left( \frac{\text{sim}(f_\theta(c), g(d_j))}{\tau} \right)}$$
- $f_\theta(c)$：模型對上下文的隱空間表徵向量。
- $g(d_i)$：工具功能說明的語義嵌入。
- $\tau$：波茲曼溫度參數（Temperature），控制決策的確定性。

### 2. 語法約束下的參數生成 (Constrained Argument Generation)
當選定工具 $t^*$ 後，參數生成為帶有上下文無關文法（Context-Free Grammar, CFG）約束 $\mathcal{G}_{\text{JSON}}$ 的自回歸生成：
$$P_\theta(\text{Args} \mid c, t^*) = \prod_{m=1}^M P_\theta(w_m \mid w_{<m}, c, t^*, \mathcal{G}_{\text{JSON}})$$
此處遮罩（Masking）機制確保生成的 Token 序列嚴格遵守 JSON Schema，避免語法報錯。

### 3. 工具執行的狀態轉移 (Environment State Transition)
執行軌跡由交互序列組成：
$$\text{Trajectory } \tau = \left( s_0, a_1, o_1, s_1, a_2, o_2, \dots, s_T \right)$$
其中 $a_t = (t^*_t, \text{Args}_t)$ 為模型動作，觀察值 $o_t = \text{Exec}(a_t)$ 為外部確定性執行函數的回傳值，模型藉由拼接上下文更新狀態：
$$s_t = [s_{t-1} \circ a_t \circ o_t]$$

---

## 實務上怎麼用？（Tools & Application）

在真實應用中，Tool Use 是建構自主代理人（[[Agentic Workflow]]）的基石：

- **終端代碼代理**：如 [[AI 程式碼代理人]]（Claude Code, Cursor Composer），其調用的工具就是 `bash`、`read_file`、`edit_file`。
- **辦公與生產力自動化**：模型調用 Google Calendar API 排定行程、調用 GitHub API 發起 PR。
- **現代最佳實踐**：強烈建議採用 **[[Model Context Protocol|MCP]]** 規格，透過現成的 MCP Server（如 SQLite、GitHub、Slack）直接為你的 AI 助手擴充工具庫，無需重複造輪子。

---

## 知識前沿與學習階梯 (Prerequisites & Frontier)

- **前置基石**：
  - `[[大語言模型-llm]]`：工具決策的核心大腦。
  - `[[Prompt Engineering]]`：如何撰寫讓模型精確理解的工具描述（Tool Description）。
- **進階延伸**：
  - `[[Model Context Protocol]] (MCP)`：工具調用生態的通用統一協議標準。
  - `[[Agentic Workflow]]`：多個工具連續調用與自主循環思考架構。
  - `[[MCP 實務安全與防護]]`：避免工具遭到惡意提示詞劫持的安全規範。
