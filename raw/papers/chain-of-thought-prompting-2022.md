# Chain-of-Thought Prompting Elicits Reasoning in Large Language Models

- **作者**：Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Fei Xia, Ed Chi, Quoc V. Le, Denny Zhou (Google Research, Brain Team)
- **發表時間**：January 2022 (NeurIPS 2022, arXiv:2201.11903)
- **原始來源**：`raw/papers/chain-of-thought-prompting-2022.pdf`

---

## 報告核心內容精煉

### 1. 核心突破：思維鏈 (Chain of Thought, CoT)
- **問題痛點**：即使像 GPT-3 這樣強大的 175B 模型，在面對多步驟數學應用題（如 GSM8K）與符號推理時，標準的 Few-shot Prompting 表現依然很差，因為模型試圖「直接從題目一次跳到答案」。
- **解決方案（思維鏈 CoT）**：
  - 在 Prompt 的 Few-shot 示範中，不直接給答案，而是刻意示範一段**「中間思考推理過程（A series of intermediate natural language reasoning steps）」**。
  - 範例：傳統 Prompt 示範 `Q: ... A: 答案是 11`；CoT 示範 `Q: ... A: 羅傑一開始有 5 個球，買了 2 罐每罐 3 個，共加 6 個，所以 5 + 6 = 11。答案是 11`。
- **效果**：在 GSM8K 算術推理測試中，GSM8K 基準準確率從標準提示的 17.9% 暴增至 **56.9%**，一舉超越了專門微調的模型。

### 2. 湧現能力與模型規模的非線性關係
- **小型模型（< 10B 參數）**：使用 CoT 毫無幫助，甚至可能產生邏輯混亂的中間文字而降低準確率。
- **大型模型（$\ge 100\text{B}$ 參數，如 PaLM 540B、GPT-3 175B）**：思維鏈能力突然「湧現」（Emergent Ability），展現極高的跳躍式成長。

### 3. 形式化定義與機率推導 (Mathematical Formulation)
- 設輸入問題為 $x$，最終答案為 $y$。
- **標準直接預測**：
  $$P_\theta(y \mid x) = \prod_{j=1}^{|y|} P_\theta(y_j \mid x, y_{<j})$$
  *(計算預測一步到位，計算圖深度受限於單次前向傳播)*。
- **思維鏈分解推理 (Chain-of-Thought Formulation)**：引入中間推理鏈 $z = (z_1, z_2, \dots, z_K)$：
  $$P_\theta(y \mid x) = \sum_{z} P_\theta(z \mid x) P_\theta(y \mid x, z) \approx \prod_{k=1}^K P_\theta(z_k \mid x, z_{<k}) \cdot \prod_{j=1}^{|y|} P_\theta(y_j \mid x, z, y_{<j})$$
  *(透過自回歸生成中間思考 Token $z$，模型在實體上**延長了前向傳播的計算步數（Test-time Compute）**，為複雜邏輯推導提供了足夠的隱空間變換容量)*。
