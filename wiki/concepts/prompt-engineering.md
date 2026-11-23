---
title: Prompt Engineering 與思維鏈 (Prompt Engineering & Chain-of-Thought)
aliases: [提示詞工程, 提示工程, 思維鏈, Chain of Thought, CoT, Few-shot Prompting]
type: concept
tags: [ai, prompt-engineering, cot, in-context-learning, reasoning]
created: 2026-11-23
updated: 2026-11-23
status: stable
sources:
  - "[[raw/papers/chain-of-thought-prompting-2022.md]]"
  - "[[raw/papers/gpt3-few-shot-learners-2020.md]]"
---

# Prompt Engineering 與思維鏈 (Prompt Engineering & Chain-of-Thought)

## 一句話白話直覺 (Intuition)
如果說 [[大語言模型-llm]] 是一個「智商極高但個性急躁的數學神童」；如果你直接問他：「小明有 5 顆蘋果，買了 3 箱每箱 4 顆，吃掉 2 顆剩幾顆？請直接給我答案！」他可能會因為急於脫口而出而算錯。**Prompt Engineering（提示詞工程）** 就是**學會如何向這位神童精準出題**；而其中最神奇的招式 **思維鏈 (Chain of Thought, CoT)** 就是對他說一句：**「請在給答案之前，先一步步列出你的思考算式（Let's think step by step）」**。神童一邊寫草稿一邊推導，原本算錯的題目瞬間全部答對！

---

## 它解決了過去什麼痛點？

### 傳統直接提示（Standard Prompting）的死角：
1. **多步推理崩潰（Failure in Multi-Step Logic）**：傳統 LLM 在處理多步驟算術題（如 GSM8K）、代碼除錯、符號邏輯時，若要求直接輸出答案，模型只有一次前向傳播的隱空間變換機會，容易「跳步」猜錯。
2. **缺乏可解釋性（Black-box Output）**：直接給出的答案若出錯，人類根本無法得知模型在哪一個邏輯節點被誤導。

### 思維鏈 (CoT) 的突破：
- **算術推理爆發性提升**：在 Google 2022 年經典實驗中，僅僅在少樣本示範中加入中間思考步驟，GSM8K 數學評測準確率從 **17.9% 暴增至 56.9%**！
- **可除錯與可審查性**：人類能清晰閱讀中間推理鏈（$z_1 \to z_2 \to \dots$），一眼看出是概念理解錯誤還是簡單計算筆誤。
- **孕育了後續的 `[[Agentic Workflow]]`**：思維鏈是代理人「ReAct 框架（Thought $\to$ Action $\to$ Observation）」的最早理論始祖。

---

## 核心機制與主流提示範式 (Core Paradigms)

```text
┌────────────────────────────────────────────────────────┐
│ 1. 標準 Few-shot Prompting (直接給答案)                │
│    Q: 停車場有 3 輛車，來了 2 輛。現在有幾輛？          │
│    A: 答案是 5。                                       │
│    Q: 羅傑有 5 個網球，買了 2 罐每罐 3 個。剩幾個？    │
│    A: 答案是 11。(容易心算錯誤)                         │
├────────────────────────────────────────────────────────┤
│ 2. Few-shot CoT (示範中間思考算式)                     │
│    Q: 羅傑有 5 個網球，買了 2 罐每罐 3 個。剩幾個？    │
│    A: 羅傑原本有 5 個。2 罐每罐 3 個即 2 * 3 = 6 個。   │
│       5 + 6 = 11。所以答案是 11。                      │
├────────────────────────────────────────────────────────┤
│ 3. Zero-shot CoT (神級啟發語)                          │
│    不給任何範例，僅在指令結尾加上：                     │
│    "Let's think step by step." (讓我們一步一步思考。)   │
└────────────────────────────────────────────────────────┘
```

---

## 形式化定義與數學基礎 (Mathematical Foundations)

### 1. 潛在推理鏈條件機率分解 (Marginal Probability Decomposition)
設輸入問題為 $x$，最終目標答案為 $y$。
- **標準提示 (Standard Direct Generation)**：模型試圖直接最大化條件機率：
  $$P_\theta(y \mid x) = \prod_{j=1}^{|y|} P_\theta(y_j \mid x, y_{<j})$$
  在此設定下，計算圖的有效深度被嚴格限制在生成答案 $y$ 的長度範圍內。
- **思維鏈提示 (Chain-of-Thought Generation)**：引入一個中間思考鏈序列變數 $\mathbf{z} = (z_1, z_2, \dots, z_K) \in \mathcal{Z}$，最終答案的邊際機率轉化為對所有可能思考路徑的全機率積分/求和：
  $$P_\theta(y \mid x) = \sum_{\mathbf{z} \in \mathcal{Z}} P_\theta(\mathbf{z} \mid x) \cdot P_\theta(y \mid x, \mathbf{z})$$
  自回歸生成過程形式化為兩階段聯合分佈：
  $$P_\theta(\mathbf{z}, y \mid x) = \underbrace{\left[ \prod_{k=1}^K P_\theta(z_k \mid x, z_{<k}) \right]}_{\text{思考推理階段 (Reasoning Phase)}} \times \underbrace{\left[ \prod_{j=1}^{|y|} P_\theta(y_j \mid x, \mathbf{z}, y_{<j}) \right]}_{\text{答案產出階段 (Answer Phase)}}$$

### 2. 測試時計算量擴展 (Test-Time Compute Scaling) 的物理意義
從計算複雜度與 Transformer 注意力機制視角來看：
- 每生成一個中間思考 Token $z_k$，自注意力矩陣（Self-Attention Matrix）就增加了一維的特徵交互維度。
- 透過人為引導模型生成長達數百個 Token 的思維鏈 $\mathbf{z}$，實質上是**在不重新訓練模型權重的前提下，動態擴大了推論時的計算量（Test-time Compute）**，為 Transformer 隱空間的多次非線性向量投影提供了充足的運算深度。

### 3. 自我一致性多數決 (Self-Consistency via Majority Voting)
為了進一步消除單一路徑的偶發性推理錯誤，Wang 等人提出了自洽性（Self-Consistency）採樣：在非零溫度（$\tau > 0$）下獨立採樣 $M$ 條不同的推理軌跡 $\{(\mathbf{z}^{(i)}, y^{(i)})\}_{i=1}^M$，並對最終答案進行多數決投票：
$$y^* = \arg\max_{y \in \mathcal{Y}} \sum_{i=1}^M \mathbb{I}\left( y^{(i)} = y \right)$$

---

## 實務上怎麼用？（Tools & Application）

1. **結構化 Prompt 最佳實踐框架（如 CRISPE 準則）**：
   - **Capacity / Role**：賦予清晰專業角色（如「你是一位擁有 10 年資歷的 Python 資深架構師」）。
   - **Insight / Context**：提供充分背景背景資料與約束條件。
   - **Statement / Goal**：清楚定義核心任務目標。
   - **Personality / Format**：指定輸出格式（如 JSON、Markdown 表格）。
2. **現代推理模型（Reasoning Models）的內化**：
   - 最新的推論模型（如 OpenAI o1/o3、DeepSeek-R1）已不再需要使用者手動輸入 "Let's think step by step"，而是在模型內部將思維鏈訓練為原生的隱式思考 Token（Thinking Tokens）。

---

## 知識前沿與學習階梯 (Prerequisites & Frontier)

- **前置基石**：
  - `[[大語言模型-llm]]`：提示詞所操控的底層自回歸模型大腦。
- **進階延伸**：
  - `[[Tool Use]]`：思維鏈加上工具調用，讓思考延伸為實際動作。
  - `[[Agentic Workflow]]`：將單次思維鏈擴展為多 Agent 循環反思與辯論。
  - `[[Vibe Coding]]`：自然語言 Prompt 在軟體開發中的終極落地形態。
