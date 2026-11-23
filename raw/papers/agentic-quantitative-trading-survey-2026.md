# Agentic Quantitative Trading: A Survey of Workflows, Systems, and Evaluation

- **作者**：Fengrui Hua, Hengyi Yang, Xinlei Hao, Haohan Zhang, Bokai Cao, Yiyan Qi, Jia Li, Jian Guo (HKUST-GZ, Peking University, IDEA Research)
- **發表時間**：August 2026 (arXiv:2608.31041v1)
- **文獻性質**：針對「代理人架構在量化交易（Agentic Quantitative Trading）」領域的首篇完整系統性綜述，涵蓋工作流、系統架構、協同機制、適應機制與評測標準。
- **原始來源**：`raw/papers/agentic-quantitative-trading-survey-2026.md`

---

## 報告核心內容精煉

### 1. 什麼是「Agentic（代理人化）」？
- **超越單次文字生成 (Beyond One-shot Generation)**：不只是輸入提示詞產出一段文字，而是具備：
  1. **任務分解 (Task Decomposition)**：將複雜目標拆解為多個中間子任務。
  2. **工具調用 (Tool Use)**：主動調用外部資料庫、回測引擎（Backtesting Engines）與 API。
  3. **狀態與記憶維護 (State & Memory)**：維持任務上下文與歷史經驗。
  4. **回饋、反思與疊代規劃 (Feedback, Reflection & Iterative Planning)**。

### 2. 量化交易全流程的五大階段 (5-Stage Pipeline)
1. **因子挖掘 (Factor Mining)**：生成式探索、多 Agent 辯論（FactorMAD）、自我演化（AlphaAgent）、工具增強（LLM-MCTS）。
2. **交易訊號發現 (Signal Discovery)**：推理型訊號、因子模型訊號（RD-Agent）、可執行策略代碼訊號。
3. **投資組合構建 (Portfolio Construction)**：資產標的篩選、權重優化（結合 Black-Litterman 與均值變異數）。
4. **訂單執行 (Order Execution)**：指令解析、演算法拆單（PACE）、確定性安全檢查（AgenticAITA）。
5. **風險管理 (Risk Management)**：事前防禦性硬門禁（SAE 冷卻期/滑點限制）、事後自適應分層記憶（FinMem）。

### 3. 多代理人系統設計的三大維度 (System Dimensions)
- **系統架構 (System Architecture)**：
  - *循序型 (Sequential)*：單向傳遞，易追溯但無法回頭。
  - *疊代型 (Iterative)*：結合回饋與記憶，循環修正策略。
  - *階層型 (Hierarchical)*：經理與分析師架構，經理指派與審核。
  - *編排型 (Orchestration)*：運用 **MCP** 與 A2A 協議控制整個工作流走向。
  - *人機協同 (Human Collaboration)*：保留人類風險偏好輸入與審核。
- **協同機制 (Coordination Mechanism)**：
  - *直接通訊 (Direct)* vs. *聚合 (Aggregation)*（將多 Agent 意見綜合）。
  - *辯論 (Debate)* vs. *篩選 (Selection)*。
  - *硬門禁 (Gating)*：嚴格 JSON 契約與確定性規則，攔截違規操作。
- **適應機制 (Adaptation Mechanism)**：
  - *記憶 (Memory)*、*反思 (Reflection)*、*獎勵 (Reward)*、*市場風格自適應 (Regime-based)*。

### 4. 關鍵實證發現與工程幻象 (Reality Check & Empirical Findings)
1. **訊號挖掘普遍，完整閉環極少**：大部分系統只會「推測漲跌」，能真正打通資產配置、下單執行與風控的端到端系統極為罕見。
2. **「回測幻象（Profit Mirage）」與數據洩漏**：
   - 很多論文在歷史回測表現驚人，但實盤（Live Market）大多虧損。原因在於 LLM 訓練資料已「記住」了歷史價格（Data Contamination），或是受到市場摩擦力（流動性不足、滑點 Slippage、手續費）的致命衝擊。
3. **系統架構比底層模型更關鍵**：
   - 實證測試顯示（Agent Market Arena）：在相同輸入下，**Agent 的協同架構設計**所解釋的績效差異，遠大於單純替換更強的 LLM 基座模型！
