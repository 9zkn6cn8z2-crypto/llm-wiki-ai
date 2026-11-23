---
title: Model Context Protocol (MCP)
aliases: [MCP, 模型上下文協議, MCP 協議]
type: concept
tags: [ai, agent, mcp, protocol, tool-use, anthropic, openai]
created: 2026-11-23
updated: 2026-11-23
status: stable
sources:
  - "[[raw/papers/mcp-landscape-security-2025.md]]"
---

# Model Context Protocol (MCP)

## 一句話白話直覺 (Intuition)
在智慧型手機剛出現時，每家廠商的充電線孔都不同（有的圓孔、有的寬扁孔、有的 Lightning），換一台手機所有線都要重買；直到 **Type-C (USB-C)** 出現，一條線搞定所有設備充電與傳輸。**MCP 就是 AI 界的 USB-C**！它讓任何 AI 模型（Claude、GPT、Gemini）只要插上遵守 MCP 規範的插頭，就能立刻操控 GitHub、查詢本地資料庫、發送 Slack 訊息，徹底告別過去每接一個工具就要寫一套專屬轉接頭的混亂局面。

---

## 它解決了過去什麼痛點？

### 傳統工具整合的「碎片化災難」：
1. **$M \times N$ 的重複造輪子**：如果有 10 種 AI 應用（Cursor, Claude, 自建 Agent）想連接 10 個外部系統（GitHub, Notion, SQLite...），傳統做法需要開發 $10 \times 10 = 100$ 次不同的串接邏輯。
2. **封閉外掛孤島 (Plugin Silos)**：2023 年 OpenAI 的 ChatGPT Plugins、字節跳動的 Coze 外掛，全都綁定在各自封閉的平台生態中，無法跨平台遷移。
3. **被動 vs. 主動的局限**：傳統 [[檢索增強生成-rag]] 只能將資料撈給模型看（唯讀），無法讓模型主動在外部系統執行操作。

### MCP 的突破：
- **開源且平台中立（Open & Platform-Agnostic）**：由 Anthropic 於 2024 年底開源推出，但迅速被 OpenAI、Cursor、JetBrains、Cloudflare 共同採納為通用行業標準。
- **動態發現與雙向互動**：模型可以動態查詢 Server 目前有哪些可用能力，並支援即時狀態通知與多步驟複雜任務。

---

## 核心架構三要素 (Architecture)

MCP 借鑒了微軟在程式碼編輯器中極為成功的 LSP（Language Server Protocol）架構，由三個核心角色組成：

```text
┌──────────────────────────────────────────────┐
│  MCP Host (宿主應用，如 Claude Desktop, Cursor) │
│  ┌────────────────────────────────────────┐  │
│  │  MCP Client (通訊中介，管理連線與取樣)    │  │
│  └───────────────────┬────────────────────┘  │
└──────────────────────┼───────────────────────┘
                       │ 傳輸層 (JSON-RPC / SSE / Stdio)
                       ▼
┌──────────────────────────────────────────────┐
│  MCP Server (提供能力的服務端)                │
│  ├── 1. Tools     : 外部可調用操作 (API, CLI)  │
│  ├── 2. Resources : 資料來源 (文件, 本地資料庫)│
│  └── 3. Prompts   : 預設重複工作流範本        │
└──────────────────────────────────────────────┘
```

### 服務端提供的三大核心能力：
1. **Tools（工具）**：給 AI 手和腳。允許模型調用特定函式執行動作（如：執行 SQL 查詢、發起 Git Commit、打電話）。
2. **Resources（資源）**：給 AI 眼睛。將外部靜態/動態資料暴露給模型讀取（如：專案程式碼目錄、即時系統 Log、PDF 文件）。
3. **Prompts（提示詞範本）**：給 AI 經驗手冊。由 Server 預先定義好的標準化工作流程或問答範本，提升重複性任務的執行效率。

---

## 形式化定義與通訊協議數學抽象 (Mathematical & Protocol Foundations)

### 1. 連接複雜度量級改善 (Complexity Reduction)
設環境中有 $M$ 個異質 LLM 客戶端（Hosts/Clients），$N$ 個外部資料源與工具系統（Servers）：
- **無統一協議時之串接邊界數**：
  $$\text{Edges}_{\text{Ad-hoc}} = M \times N \in \mathcal{O}(M \cdot N)$$
- **MCP 雙邊解耦標準化後之邊界數**：
  $$\text{Edges}_{\text{MCP}} = M + N \in \mathcal{O}(M + N)$$
  *(以漸進複雜度解釋了為何 MCP 能引爆生態系規模效應)*。

### 2. 基於 JSON-RPC 2.0 的雙向通訊狀態機
MCP 訊息交換形式化為有向狀態轉移元組 $\langle \text{ID}, \text{Method}, \text{Payload} \rangle$：
- **能力發現 (Capability Handshake)**：
  $$\mathcal{C}(\text{Server}_j) = \langle \mathcal{T}_j, \mathcal{R}_j, \mathcal{P}_j \rangle$$
  客戶端在初始化階段聚合全域能力空間：$\mathcal{T}_{\text{global}} = \bigcup_{j=1}^N \mathcal{T}_j$。
- **工具調用決策最佳化 (Tool Selection Metric)**：
  在使用者請求 $q$ 到達時，客戶端透過加權相似度與可信度函數選取目標工具 $t^*$：
  $$t^* = \arg\max_{t \in \mathcal{T}_{\text{global}}} \left[ \alpha \cdot \cos\left( \mathbf{e}_q, \mathbf{e}_{d(t)} \right) + (1-\alpha) \cdot \text{Trust}(t) \right]$$
  其中 $\mathbf{e}_q, \mathbf{e}_{d(t)}$ 分別為請求與工具說明的嵌入向量（Embedding），$\text{Trust}(t) \in [0, 1]$ 為認證與沙盒可信評分。

---

## 實務上怎麼用？（Tools & Application）

目前 MCP 已成為現代 AI 應用的標配協議，主流生態包含：

1. **主流客戶端支援 (MCP Clients / Hosts)**：
   - **Claude Desktop**：原生支援在配置檔加入任何本機或遠端 MCP Server。
   - **Cursor / Windsurf**：在次世代 IDE 中透過 MCP 讓 AI 深入存取終端、資料庫與自定義內部工具。
   - **OpenAI Agent SDK**：OpenAI 官方 SDK 已全面原生整合 MCP。
2. **社群開源 Servers (隨插即用)**：
   - 社群已湧現數千個現成 Server（可於 `mcp.so`、`glama.ai` 探索）：涵蓋 GitHub、PostgreSQL、Puppeteer 瀏覽器自動化、Slack、Google Drive 等。
3. **雲端託管模式 (Remote MCP)**：
   - 藉由 Cloudflare 遠端託管與 OAuth 認證，讓企業不必在本地運行 Server，實現跨裝置、多租戶的安全調用。

---

## 知識前沿與學習階梯 (Prerequisites & Frontier)

- **前置基石**：
  - `[[Tool Use]]`：MCP 所標準化的核心概念（工具調用本質）。
  - `[[大語言模型-llm]]`：驅動決策的核心模型。
- **進階延伸**：
  - `[[MCP 實務安全與防護]]`：工具流劫持、提示詞欺騙與沙盒逃逸防範。
  - `[[AI 程式碼代理人]]`：Cursor 與 Claude 如何借助 MCP 成為全功能工程師。
  - `[[Agentic Workflow]]`：多個 MCP 工具協同串接的自主代理人工作流。
