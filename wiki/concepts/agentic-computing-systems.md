---
title: 代理人運算系統與硬體架構 (Agentic Computing Systems & Hardware Architecture)
aliases: [Agentic AI 算力架構, 代理人硬體架構, Agentic Systems & Infra]
type: concept
tags: [ai, agent, hardware, systems, datacenter, cpu, gpu, microarchitecture]
created: 2026-11-23
updated: 2026-11-23
status: stable
sources:
  - "[[raw/papers/architectural-implications-agentic-ai-2026.md]]"
---

# 代理人運算系統與硬體架構 (Agentic Computing Systems & Hardware Architecture)

## 一句話白話直覺 (Intuition)
跑傳統的單體 LLM（如 ChatGPT 網頁對話）就像**「開重型卡車跑直線高速公路」**：一踩油門，主要全靠 GPU 這顆大引擎進行密集的矩陣乘法，旁邊的副駕駛（CPU）只負責遞水；然而跑 **Agentic AI** 卻像**「在擁擠的市區反覆走走停停、倒車入庫、卸貨搬磚」**：AI 剛用 GPU 推理了兩句，就要跳回主機 CPU 跑腳本、查資料庫、執行編譯，然後再跳回 GPU。兩者之間頻繁「打乒乓球」，使得原本默默無聞的副駕駛（CPU）瞬間成了最容易塞車的交通瓶頸！

---

## 它解決了過去什麼痛點？

### 傳統雲端與 AI 伺服器設計的重大失配：
1. **CPU 被邊緣化的假設破滅**：過去業界認為「AI 時代只要買最貴的 GPU，CPU 隨便配個便宜的就行」。但在 Agentic 任務中，超過 **27% 的請求時間花在 CPU 工具調用與狀態編排上**，CPU 重新回到了整體延遲的**關鍵路徑 (Critical Path)**。
2. **算力擱置與浪費 (Stranded Capacity)**：
   - 傳統靜態分配資源：平時 CPU/GPU 平均使用率很低（CPU 僅 6%~31%，GPU SM 算力使用率低於 55%）。
   - 但每當 Agent 觸發複雜工具（如代碼測試或全庫檢索）時，CPU 瞬間突發衝破 100% 飽和，造成「平時閒死、突發卡死」的嚴重資源擱置。
3. **同質化核心配置引發微架構踩踏**：將不同特性的 Agent 角色混在相同的 CPU 核心池，造成處理器快取（Cache）互相清洗淘汰，流水線嚴重卡頓停頓（Pipeline Stalls）。

---

## 核心機制與三大軟體角色 (Core Mechanisms & Software Roles)

在底層生產系統（如微軟 Azure 雲端艦隊）中，Agentic 系統被解構為三種資源特徵截然不同的軟體角色：

```text
       ┌────────────────────────────────────────────────────────┐
       │                   Host (主機端 CPU)                    │
       │                                                        │
       │  ┌──────────────────┐            ┌──────────────────┐  │
       │  │ 1. 調度器        │            │ 2. 編排器        │  │
       │  │ (Schedulers)     │            │ (Orchestrators)  │  │
       │  │ • 延遲極度敏感   │            │ • 追蹤 DAG 狀態  │  │
       │  │ • 分發推論請求   │            │ • 路由 Agent 訊息│  │
       │  └────────┬─────────┘            └────────┬─────────┘  │
       │           │                               │            │
       │           │      ┌──────────────────┐     │            │
       │           │      │ 3. 執行器        │     │            │
       │           │      │ (Runners)        │ ◄───┘            │
       │           │      │ • 運算密集/突發  │                  │
       │           │      │ • 執行工具腳本   │                  │
       │           │      └──────────────────┘                  │
       └───────────┼────────────────────────────────────────────┘
                   │ 跨 CPU-GPU 邊界 (頻繁交互)
                   ▼
       ┌────────────────────────────────────────────────────────┐
       │                 Accelerator (GPU / NPU)                │
       │ • 負責自回歸生成與 Dense Tensor 矩陣計算              │
       └────────────────────────────────────────────────────────┘
```

1. **Schedulers（調度器）**：負責向 GPU 引擎分發請求。負載小但延遲極度敏感，一旦受阻會造成 GPU 全面斷炊。
2. **Orchestrators（編排器）**：維護任務有向無環圖（DAG）狀態，管理訊息傳遞。
3. **Runners（執行器）**：實際在作業系統中運行 Bash、檔案 I/O、編譯器與外部 API。具有極高波動性與突發算力需求。

---

## 形式化定義與微架構數學基礎 (Mathematical Foundations)

### 1. 任務端到端時延分解模型 (End-to-End Latency Decomposition)
一個包含 $N$ 次模型推論與 $M$ 次工具調用的 Agentic 請求，其整體時延 $T_{\text{total}}$ 嚴格形式化為：
$$T_{\text{total}} = \sum_{i=1}^{N} T_{\text{GPU}}^{\text{infer}}(i) + \sum_{j=1}^{M} T_{\text{CPU}}^{\text{tool}}(j) + \sum_{k=1}^{K} T_{\text{host}}^{\text{overhead}}(k)$$
其中 $T_{\text{host}}^{\text{overhead}}$ 包含進程上下文切換、記憶體狀態序列化與跨硬體資料搬移開銷。
- 當系統出現工具耗時長尾分佈（Heavy-tail distribution）時：
  $$\mathbb{P}\left( \sum_{j=1}^M T_{\text{CPU}}^{\text{tool}}(j) > \sum_{i=1}^N T_{\text{GPU}}^{\text{infer}}(i) \right) \ge 0.27$$
  這在數學上嚴格證明了：**單純縮減 GPU 推論時間 $T_{\text{GPU}}$ 對總時延的邊際貢獻已呈現遞減（Amdahl's Law），優化主機 CPU 鏈路成為瓶頸破局點**。

### 2. 微架構效能指標 Top-Down 分解 (Intel/AMD TMA Formulation)
衡量主機 CPU 執行效率的每週期指令數（IPC）與流水線槽（Pipeline Slots）形式化為：
$$\text{IPC} = \frac{\text{Retired Instructions}}{\text{CPU Cycles}}$$
$$\text{Pipeline Slots} = \text{Retiring} + \text{Frontend Bound} + \text{Backend Bound} + \text{Bad Speculation}$$
- **實證測量發現**：在多 Agent 混部環境中，$\text{Retiring}$（有效執行）佔比極低，而 $\text{Backend Bound}$（因等待記憶體與 L3 快取失效停頓）高達 **43% ~ 47%**，快取每千條指令失效數（L1-Data MPKI）飆升至 14~20。

---

## 實務上怎麼用？（Tools & Application）

理解 Agentic 算力架構對開發者與架構師的實務指引：

1. **本機開發與地端部署**：
   - 部署 Coding Agent（如 Cursor、Claude Code）時，主機 CPU 的單核時脈與 L3 快取大小至關重要，不可忽視。
2. **次世代雲端系統調優**：
   - 採納如 **[[Agentic 推論服務與硬體調優實踐|Agora]]** 的設計範式：將調度器核心獨立隔離、動態收割閒置 CPU、並依任務進行 CPU 核心綁定（Task Pinning）。

---

## 知識前沿與學習階梯 (Prerequisites & Frontier)

- **前置基石**：
  - `[[Agentic Workflow]]`：軟體層面的多代理人工作流本質。
  - `[[Tool Use]]` 與 `[[Model Context Protocol]]`：頻繁引發 CPU-GPU 交互的通訊協議。
- **進階延伸**：
  - `[[Agentic 推論服務與硬體調優實踐]]`：微軟 Azure 與學界提出的 Agora 系統落地解法。
