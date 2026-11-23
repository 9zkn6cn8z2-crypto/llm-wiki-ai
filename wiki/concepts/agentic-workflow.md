---
title: 代理人工作流與多智能體協作 (Agentic Workflow & Multi-Agent Systems)
aliases: [Agentic Workflow, 多智能體系統, MAS, 代理人架構, 思考行動迴圈]
type: concept
tags: [ai, agent, multi-agent, workflow, reasoning, orchestration]
created: 2026-11-23
updated: 2026-11-23
status: stable
sources:
  - "[[raw/papers/agentic-quantitative-trading-survey-2026.md]]"
---

# 代理人工作流與多智能體協作 (Agentic Workflow & Multi-Agent Systems)

## 一句話白話直覺 (Intuition)
傳統的 LLM 就像一個「單獨關在房間回答問題的學者」，無論問題多複雜，他只能一口氣回答完畢（單次生成），錯了就全盤皆錯；而 **Agentic Workflow（代理人工作流）** 則是**組建一個由多位專家組成的虛擬團隊**——有的負責將任務拆解成待辦清單（規劃者 Planner）、有的負責調用終端機與軟體（執行者 Executor）、有的專門挑刺反駁（辯論審查者 Critic），最後還有品管經理進行規則把關（Gatekeeper），在不斷「思考 $\to$ 行動 $\to$ 反思 $\to$ 修正」的迴圈中完成長鏈條的複雜任務。

---

## 它解決了過去什麼痛點？

### 傳統單次生成（One-shot Generation）的瓶頸：
1. **長任務崩潰（Horizon Collapse）**：當任務步驟超過 5 步時，單一 LLM 容易在中間步驟產生幻覺並自我偏離，最終無法達成原定目標。
2. **缺乏自我糾錯機制（No Self-Correction）**：傳統 LLM 無法觀察自己的輸出在真實世界中是否成功，錯了也不自知。
3. **視角單一與盲點**：單一提示詞容易陷入單一思維定勢，缺乏多角審查與制衡。

### Agentic 模式的突破：
- **目標遞歸拆解（Goal Decomposition）**：將抽象的大目標（如「開發一個量化系統」）分解為離散的子目標。
- **動態環境互動與狀態追蹤**：結合 `[[Tool Use]]` 與 `[[Model Context Protocol|MCP]]` 主動與外部環境互動。
- **多角度辯論與硬門禁（Debate & Gating）**：引入多角色交叉質詢，並以確定性程式碼規則進行安全攔截。

---

## 核心機制與架構維度 (System Dimensions)

根據 2026 年系統性學術綜述，一個完整的 Agentic 系統由三大維度構成：

```text
┌───────────────────────┐   ┌───────────────────────┐   ┌───────────────────────┐
│ 1. 系統架構 (Arch)    │   │ 2. 協同機制 (Coord)   │   │ 3. 適應機制 (Adapt)   │
│ - 循序型 (Sequential) │   │ - 直接傳遞 (Direct)   │   │ - 分層記憶 (Memory)   │
│ - 疊代型 (Iterative)  │   │ - 意見聚合 (Aggregate)│   │ - 事後反思 (Reflect)  │
│ - 階層型 (Hierarch)   │   │ - 交叉辯論 (Debate)   │   │ - 績效回饋 (Reward)   │
│ - 編排型 (Orchestrate)│   │ - 確定性門禁 (Gating) │   │ - 模式自適應 (Regime) │
└───────────────────────┘   └───────────────────────┘   └───────────────────────┘
```

### 1. 系統架構類型 (System Architecture)
- **循序型 (Sequential Pipeline)**：$A \to B \to C$ 的流水線，容易排查日誌，但後續節點無法要求前續重構。
- **疊代型 (Iterative Loop)**：具備反饋通道，執行失敗後透過記憶重新發起第 $t+1$ 輪嘗試。
- **階層型 (Hierarchical)**：主管 Agent 負責派發任務給專業分析師，並做最終裁決。
- **編排型 (Orchestration)**：由總調度器透過 **[[Model Context Protocol|MCP]]** 或 A2A（Agent-to-Agent）協議動態調派工具與 Agent。

### 2. 協同決策機制 (Coordination Mechanism)
- **辯論 (Debate)**：多個具備不同偏見或角色的 Agent 互相質詢，逼出隱藏邏輯錯誤。
- **確定性門禁 (Hard Gating)**：在 AI 做出決策與真實下單/執行之間，配置嚴格的型別合約（Typed JSON）與安全程式碼規則，不合規者強制阻斷。

---

## 數學抽象與形式化基礎 (Mathematical Foundations)

在學術與系統工程中，Agentic Workflow 不只是對話，而是被嚴格抽象為**部分可觀察馬可夫決策過程 (POMDP)** 或動態信念更新模型：

### 1. 代理人序列決策形式化 (POMDP Formulation)
Agent 系統運作在一個元組 $\mathcal{M} = \langle \mathcal{S}, \mathcal{A}, \mathcal{T}, \mathcal{R}, \Omega, \mathcal{O}, \gamma \rangle$ 之上：
- **環境狀態 $s_t \in \mathcal{S}$**：外部真實世界的完整狀態（例如整個程式碼庫或市場真實訂單簿）。
- **觀察 $o_t \in \Omega$**：Agent 透過環境回傳所看到的局部資訊，由觀察機率函數產生：
  $$o_t \sim \mathcal{O}(s_t)$$
- **動作 $a_t \in \mathcal{A}$**：包含**自然語言推理 (Thought)**、**工具調用 (Tool Call)** 與 **多 Agent 訊息交換 (Communication)**。
- **狀態轉移 $\mathcal{T}$**：外部環境依動作演進：$s_{t+1} \sim \mathcal{T}(s_t, a_t)$。

### 2. 反思與信念更新 (Bayesian Belief Updating via Reflection)
當 Agent $i$ 收到執行結果或外部批評反饋 $F_t$ 時，其內部記憶與決策信念 $\theta$ 進行遞歸更新：
$$P(\theta_{t+1} \mid F_{1:t}) = \frac{P(F_t \mid \theta_t) \cdot P(\theta_t \mid F_{1:t-1})}{P(F_t \mid F_{1:t-1})}$$
在實務中，這表現為 Agent 將歷史失敗經驗（Trajectory）存入向量或分層記憶中，在下一輪決策時動態檢索並作為 In-Context 條件先驗。

---

## 實務上怎麼用？（Frameworks & Application）

目前構建 Agentic Workflow 的主流開源與商用架構：
1. **開發框架**：
   - **LangGraph**：以有向無環圖（DAG）與循環圖精確控制 Agent 的狀態機與分支邏輯。
   - **AutoGen / CrewAI**：快速定義不同角色的虛擬團隊（如 Product Manager, Coder, Reviewer）進行會話式協同。
   - **OpenAI Agent SDK / Anthropic MCP**：基於底層通訊協議的模組化 Agent 構建。
2. **代表性前沿應用**：
   - **軟體工程**：`[[AI 程式碼代理人]]` 透過「編寫 $\to$ 測試 $\to$ 報錯反思 $\to$ 修復」循環自主提交 PR。
   - **複雜決策**：`[[代理人量化交易實踐]]` 實現因子挖掘、訊號篩選、組合構建到風控的端到端管線。

---

## 知識前沿與學習階梯 (Prerequisites & Frontier)

- **前置基石**：
  - `[[大語言模型-llm]]`：各代理人背後的基座大腦。
  - `[[Tool Use]]`：Agent 能夠實際發起動作（Action）的關鍵機制。
  - `[[Model Context Protocol]]`：Agent 與工具、Agent 與 Agent 之間通訊的標準介面。
- **進階延伸**：
  - `[[AI 程式碼代理人]]`：Agentic Workflow 在軟體開發領域的極致實踐。
  - `[[代理人量化交易實踐]]`：Agentic Workflow 在高風險金融決策中的實務系統與風控。
  - `[[代理人運算系統與硬體架構]]`：多 Agent 工作流在資料中心與主機 CPU-GPU 上的底層硬體開銷與算力架構。
