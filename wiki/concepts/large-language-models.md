---
title: 大語言模型 (Large Language Models, LLM)
aliases: [LLM, 大模型, 大型語言模型, GPT-3, 基礎模型, Foundation Models]
type: concept
tags: [ai, llm, foundation-model, gpt, scaling-laws, in-context-learning]
created: 2026-11-23
updated: 2026-11-23
status: stable
sources:
  - "[[raw/papers/gpt3-few-shot-learners-2020.md]]"
---

# 大語言模型 (Large Language Models, LLM)

## 一句話白話直覺 (Intuition)
如果把傳統的 AI 比喻為「只能在一項特定科目（例如單獨學英翻中、單獨做垃圾郵件分類）拿高分的偏科生」；那麼 **大語言模型 (LLM)** 就像一個**「讀破全人類圖書館所有文字的超級百科全書天才」**。它的核心基本功其實只有一招——**「文字接龍（預測下一個單詞）」**，但當它背下的資料達到數千億字、腦神經網絡擴大到千億個參數時，質變引發了量變，它突然「開竅（湧現）」，學會了常識推理、翻譯、寫程式甚至理解人類意圖。

---

## 它解決了過去什麼痛點？

### 傳統自然語言處理（NLP）的微調困局：
1. **任務孤島與微調成本（The Pre-train + Fine-tune Bottleneck）**：在 GPT-3 出現前，想要 AI 幫你做法律文件摘要，你必須收集數千份人工標註的法律摘要數據，去修改微調（Fine-tune）模型的所有權重。換一個任務，就必須重訓一個新模型，維護極其痛苦。
2. **缺乏常識與靈活性**：小模型容易死背標註數據的句型，稍微換個說法就無法理解。

### LLM 的典範轉移：
- **上下文學習 (In-Context Learning, ICL)**：**不需要更新模型任何權重！** 只要在輸入對話框（Context Window）中用自然語言描述規則，或給予 1~3 個範例，模型就能「在推論瞬間」立刻理解新任務並準確完成。
- **單一通用大腦**：同一個模型，既能寫詩、寫 Python 代碼、診斷醫學常識，又能作為未來 `[[Agentic Workflow]]` 的決策大腦。

---

## 核心機制與任務設定 (Core Mechanisms)

在 GPT-3 奠定的現代 LLM 範式中，任務互動分為三種基本形態：

```text
┌────────────────────────────────────────────────────────┐
│ 1. Zero-shot (零樣本學習)                              │
│    僅提供任務指令，不給任何範例：                       │
│    "請將以下繁體中文翻譯成法文：早安。" ──► "Bonjour."   │
├────────────────────────────────────────────────────────┤
│ 2. One-shot (單樣本學習)                               │
│    提供任務指令 + 1 個示範樣例：                        │
│    "早安 -> Bonjour; 晚安 -> "         ──► "Bonsoir."   │
├────────────────────────────────────────────────────────┤
│ 3. Few-shot (少樣本學習 / In-Context Learning)          │
│    在提示詞內提供 2~100 個示範樣例，引導模型捕捉特定格式│
└────────────────────────────────────────────────────────┘
```

---

## 形式化定義與數學基礎 (Mathematical Foundations)

### 1. 自回歸語言建模與交叉熵損失 (Autoregressive Loss Formulation)
設文字序列由 Token 組成：$\mathbf{x} = (x_1, x_2, \dots, x_T)$。大語言模型本質上是在估計整個序列的聯合機率分佈，根據機率鏈式法則（Chain Rule of Probability）分解為自回歸條件機率的乘積：
$$P_\theta(\mathbf{x}) = \prod_{t=1}^T P_\theta(x_t \mid x_1, x_2, \dots, x_{t-1}) = \prod_{t=1}^T P_\theta(x_t \mid x_{<t})$$
在訓練階段，給定巨量語料庫 $\mathcal{D}$，模型的優化目標是最小化**負對數似然（Negative Log-Likelihood）**，亦即標準的交叉熵損失：
$$\mathcal{L}_{\text{AR}}(\theta) = - \frac{1}{T} \sum_{t=1}^T \log P_\theta(x_t \mid x_{<t})$$
- 每一層的神經網絡狀態透過 Softmax 層投影為詞表上的機率分佈：
  $$P_\theta(x_t = w \mid x_{<t}) = \frac{\exp\left( \frac{\mathbf{h}_t^T \mathbf{w}_w}{\tau} \right)}{\sum_{w' \in \mathcal{V}} \exp\left( \frac{\mathbf{h}_t^T \mathbf{w}_{w'}}{\tau} \right)}$$
  其中 $\mathbf{h}_t$ 為 Transformer 頂層隱狀態，$\mathbf{w}_w$ 為目標單詞的輸出嵌入向量，$\tau$ 為採樣溫度。

### 2. 縮放定律 (Scaling Laws)
Kaplan 與 OpenAI 研究團隊在數學上證實，當架構不受形狀限制時，交叉熵損失 $\mathcal{L}$ 與參數量 $N$、訓練數據量 $D$ 以及總計算浮點數 $C \approx 6ND$ 遵循精確的**冪律分佈（Power Law）**：
$$\mathcal{L}(N, D) = \left( \frac{N_c}{N} \right)^{\alpha_N} + \left( \frac{D_c}{D} \right)^{\alpha_D} + \mathcal{L}_0$$
- $\alpha_N \approx 0.076, \alpha_D \approx 0.057$ 為冪律指數。
- $\mathcal{L}_0$ 為不可約減的資料熵基線。
- **物理意義**：這條公式從數學上宣告了，只要持續以最佳比例增加模型參數與訓練資料，模型的困惑度（Perplexity）與下游任務錯誤率就會呈現確定性的持續下降，奠定了現代大型模型軍備競賽的理論基石。

---

## 實務上怎麼用？（Tools & Application）

1. **主流基礎模型**：
   - 閉源 SOTA：GPT-4o、Claude 3.5 Sonnet / [[Claude Opus 4.5]]、Google Gemini 1.5/2.0。
   - 開源權重：Llama 3 系列、DeepSeek 系列、Mistral 系列。
2. **核心調用參數**：
   - **Temperature（溫度 $\tau$）**：控制輸出隨機度。寫代碼或嚴謹推理時設低（$\tau \approx 0.0 \sim 0.2$）；創意寫作時設高（$\tau \approx 0.7 \sim 1.0$）。
   - **Top-p (Nucleus Sampling)**：動態截斷累積機率達 $p$ 的 Token 候選池，兼顧穩定度與多樣性。

---

## 知識前沿與學習階梯 (Prerequisites & Frontier)

- **前置基石**：
  - `[[Transformer 架構]]`：所有現代 LLM 底層不可動搖的神經網絡核心。
- **進階延伸**：
  - `[[Prompt Engineering]]`：如何透過提示詞與思維鏈最大化激發 LLM 的推理潛能。
  - `[[Tool Use]]`：讓 LLM 走出純文字對話、獲得操作真實世界工具的能力。
  - `[[Agentic Workflow]]`：將 LLM 作為決策中樞組建自主智慧體團隊。
