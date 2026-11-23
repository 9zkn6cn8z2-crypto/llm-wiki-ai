# Architectural Implications of Agentic AI Workflows

- **作者**：Jirong Yang, Peizhe Liu, Chaojie Zhang, Jovan Stojkovic (The University of Texas at Austin, Microsoft Azure)
- **發表時間**：August 2026 (arXiv:2608.04458v1 [cs.AI])
- **文獻性質**：全球首篇針對「代理人 AI（Agentic AI）在超大規模雲端資料中心（Azure）生產環境與伺服器硬體架構層面」的系統性實證測量與架構設計論文。
- **原始來源**：`raw/papers/architectural-implications-agentic-ai-2026.md`

---

## 報告核心內容精煉

### 1. 核心洞察：Agentic AI 顛覆了傳統伺服器架構假設
- **傳統單體 LLM 推論**：Accelerator-bound（加速器/GPU 主導）。主機 CPU 僅負責分發請求，GPU 負責大批量的 Dense Tensor 矩陣運算。
- **Agentic AI 執行特徵**：
  - **高度碎片化與跨 CPU-GPU 邊界 (Fragmented & Heterogeneous)**：單一使用者請求會展開為「多次 LLM 推論 $\leftrightarrow$ 主機工具調用 $\leftrightarrow$ 編排狀態更新」的有向無環圖（DAG）。
  - **CPU 重新進入關鍵路徑 (Critical Path)**：因為編排與工具（Python、Bash、檔案讀寫、編譯）全在 Host CPU 上運行，CPU 延遲直接主導了整體任務端到端時間（超過 27% 的請求中，工具耗時超過推論耗時）。
  - **突發性與算力擱置 (Stranded Capacity)**：平時 CPU/GPU 負載極低（中位數 CPU 僅 6%~31%，GPU SM < 55%），但在工具觸發時瞬間暴衝接近 100% 飽和。

### 2. 三大軟體角色打破同質硬體配置
主機端存在三種不同特性的軟體角色：
1. **Schedulers（調度器）**：負責向 GPU 分發模型請求（延遲敏感、CPU 輕量、網路吞吐高）。
2. **Orchestrators（編排器）**：負責在 Agent 間路由狀態與訊息（延遲敏感、邏輯判斷）。
3. **Runners（執行器）**：負責實際調用與執行工具（吞吐導向、運算密集、高度突發）。
- **硬體失配後果**：若將這些角色混雜在同質 CPU 核心池，短暫且龐大的 Runner 工具突發會排擠調度器核心，且造成大量微架構快取淘汰（L1/L2 Cache MPKI 高達 14~20，IPC 僅 1.2~1.6，上下文切換從每秒 71 次暴增至 660 次）。

### 3. Agora：為 Agentic AI 量身打造的伺服器架構設計
論文提出了 Agora 原型系統，提出三大解決原則：
1. **CPU 擱置週期動態收割 (Harvesting Stranded CPU Cycles)**：
   - 角色隔離：保護延遲敏感的 Scheduler/Orchestrator 核心絕不被收割。
   - 協同編排提示 (Orchestrator Hints)：由編排器預判何時有計算型工具突發，進行漸進式優雅退讓（Adaptive Retreat）。
   - 成果：回收 95% 閒置 CPU 吞吐量，主機 CPU 使用率提升 31%，而 Agent 延遲僅增加 < 3%。
2. **GPU 擱置容量整併與超賣 (Harvesting Stranded GPU via Consolidation)**：
   - 透過混合整數線性規劃 (MILP) 動態求解最佳 Agent 放置與張量並行（Tensor Parallelism）。
   - 共享模型權重與 KV-Cache 前綴，減少 33%~50% GPU 佔用，吞吐提升 82%~106%，長尾延遲降低 2.5~3.8 倍。
3. **角色感知核心池化與任務綁核 (Role-Aware CPU Pooling & Task Pinning)**：
   - 依「任務（Task）」而非「工具類型」進行 CPU 核心綁定（Pinning），保護快取與分支預測局部性。

### 4. 關鍵數學形式化 (Mathematical Formulations)
- **微架構 Pipeline 插槽 Top-Down 分解**：
  $$\text{Pipeline Slots} = \text{Retiring} + \text{Frontend Bound} + \text{Backend Bound} + \text{Bad Speculation}$$
- **GPU 放置與並行最佳化之混合整數線性規劃 (MILP)**：
  $$\min_{\mathbf{x}, \mathbf{y}} \sum_{g \in \mathcal{G}} \text{Cost}(g) \cdot y_g + \lambda \sum_{a \in \mathcal{A}} \text{TailLatency}(a)$$
  受限於 GPU 顯存容量 $\sum_{a \in \mathcal{A}_g} \text{Memory}(a) \le M_g$ 與張量並行通訊開銷。
