# Language Models are Few-Shot Learners (GPT-3)

- **作者**：Tom B. Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, Sandhini Agarwal, Ariel Herbert-Voss, Gretchen Krueger, Tom Henighan, Rewon Child, Aditya Ramesh, Daniel M. Ziegler, Jeffrey Wu, Clemens Winter, Christopher Hesse, Mark Chen, Eric Sigler, Mateusz Litwin, Scott Gray, Benjamin Chess, Jack Clark, Christopher Berner, Sam McCandlish, Alec Radford, Ilya Sutskever, Dario Amodei (OpenAI)
- **發表時間**：July 2020 (NeurIPS 2020, arXiv:2005.14165)
- **原始來源**：`raw/papers/gpt3-few-shot-learners-2020.pdf`

---

## 報告核心內容精煉

### 1. 核心突破：In-Context Learning (上下文學習) 與湧現
- **過去典範（Pre-train + Fine-tune）**：在大型語料庫預訓練後，必須針對不同特定任務（如情緒分析、翻譯、摘要）收集數千筆標註資料並更新模型所有權重（Fine-tuning）。
- **GPT-3 的突破（In-Context Learning）**：
  - 參數量達到 175B（1,750 億參數），模型不更新任何權重，僅透過在輸入 Prompt 中給予任務說明與少數範例（Few-shot Demonstration），就能直接學會執行新任務。
  - 任務設定分為三種：
    1. **Zero-shot**：僅提供任務自然語言描述，不給任何範例。
    2. **One-shot**：提供任務描述 + 1 個示範樣例。
    3. **Few-shot**：在上下文窗口中提供 10~100 個示範樣例（典型為 $K \in [10, 100]$）。

### 2. 核心架構與縮放定律 (Scaling Laws)
- **架構基礎**：基於 Transformer Decoder-only 架構，融入 Alternating Dense and Locally Banded Sparse Attention Patterns。
- **縮放規律 (Scaling)**：
  - 橫跨 8 個模型尺度（從 125M 到 175B 參數）。
  - 跨任務效能與模型容量（Parameters）、訓練資料量（Tokens）與計算量（Compute FLOPs）呈現嚴格的冪律關係（Power Law Scaling）：
    $$\mathcal{L}(N) \approx \left( \frac{N_c}{N} \right)^{\alpha_N}$$
  - 隨著參數規模突破百億至千億級，Few-shot 準確率呈現陡峭的非線性上升（湧現能力 Emergence）。

### 3. 自回歸 Next-Token 預測與損失函數 (Mathematical Formulation)
- 目標是在給定前續詞序列 $x_{<t}$ 條件下，最大化對數似然機率（Cross-Entropy Loss）：
  $$\mathcal{L}_{\text{AR}}(\theta) = -\sum_{t=1}^T \log P_\theta(x_t \mid x_{<t})$$
  $$P_\theta(x_t \mid x_{<t}) = \text{Softmax}\left( \frac{\mathbf{h}_T \mathbf{W}_U^T}{\tau} \right)$$
- 透過極簡的「預測下一個單詞」無監督目標，在 300B Tokens 的巨量網路語料（Common Crawl, WebText2, Books, Wikipedia）上訓練，竟內化了常識推理、翻譯、算術與代碼生成能力。
