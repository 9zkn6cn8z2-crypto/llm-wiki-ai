# LLM Wiki 操作日誌 (Log)

> 本日誌為嚴格時序追加（Append-only），記錄所有 Ingest、Lint、重大 Query 與回流操作。
> 格式標頭：`## [YYYY-MM-DD] <action> | <title>`

---

## [2026-11-23] init | 建立 AI 發展與實際運用 LLM Wiki 知識庫骨幹
- 初始化專案目錄架構（`raw/` 原始資料層、`wiki/` 活體知識庫層）。
- 建立 `AGENTS.md`：定義繁體中文為主、Obsidian 雙鏈整合、AI 領域特化規則與逐步對話引導協議。
- 建立 `index.md` 與 `log.md`。

## [2026-11-23] ingest | Microsoft AI Diffusion: Q1 2026 Trends and Insights
- 依據「聚焦技術與應用、過濾宏觀總經」原則，從報告中精煉 AI Coding 與 Agentic Workflow 變革。
- 新增概念頁：`wiki/concepts/ai-coding-agents.md`、`wiki/concepts/vibe-coding.md`。
- 新增實踐頁：`wiki/practices/ai-assisted-software-engineering.md`。
- 新增實體頁：`wiki/entities/github-copilot.md`。
- 重構 `AGENTS.md`：確立零基礎友善、生活化直覺導向與知識缺口探測準則。
- 重構 `index.md`：建立「從 0 到 1：AI 技術演進階梯圖 (AI Learning Roadmap)」，標記四個階梯與未建頁知識缺口。

## [2026-11-23] lint | 確立「來源溯源與防幻覺鐵律」至 AGENTS.md
- 依據使用者指導，杜絕 Agent 擅自生成來路不明資料的風險。
- 在 `AGENTS.md` 正式加入第 6 節「來源溯源與防幻覺鐵律 (Strict Grounding & Provenance)」。
- 明訂 Wiki 頁面必須具備 `raw/` 來源依據；補齊未建頁知識缺口時，必須先由使用者提供資料，或由 Agent 提議權威來源清單經使用者審核入庫後，方可啟動 Ingest。

## [2026-11-23] ingest | Model Context Protocol (MCP): Landscape, Security Threats, and Future Research Directions
- 依據 2025 年 ACM 首篇 MCP 學術綜述論文，補齊「階梯 3：自主行動與代理」核心拼圖。
- 新增概念頁：`wiki/concepts/tool-use.md`（白話解構大腦裝上機械手、Function Calling 至 MCP 演進）。
- 新增概念頁：`wiki/concepts/model-context-protocol.md`（AI 界的 USB-C、Host/Client/Server 架構、Tools/Resources/Prompts 三大能力）。
- 新增實踐頁：`wiki/practices/mcp-security-and-safety.md`（建立/運行/更新三階段威脅、工具流劫持、沙盒逃逸防範）。
- 更新 `index.md`：在技術階梯圖中正式點亮 `[[Tool Use]]` 與 `[[Model Context Protocol]]`，並追加文獻溯源。

## [2026-11-23] ingest | Agentic Quantitative Trading: A Survey of Workflows, Systems, and Evaluation
- 依據 2026 年最新學術綜述，正式補齊「階梯 3：自主行動與代理」核心拼圖。
- 新增概念頁：`wiki/concepts/agentic-workflow.md`（虛擬專案團隊譬喻、POMDP 數學形式化、貝葉斯信念更新、四大架構與協同機制）。
- 新增實踐頁：`wiki/practices/agentic-quantitative-trading.md`（五階段量化管線、均值變異數與 Black-Litterman 數學模型、夏普/回撤指標、回測幻象破解與硬門禁防護）。
- 更新概念頁：`wiki/concepts/ai-coding-agents.md`（連結 Agentic 閉環範式）。
- 更新 `index.md`：在技術階梯圖中正式點亮 `[[Agentic Workflow]]` 與 `[[代理人量化交易實踐]]`。

## [2026-11-23] ingest | Architectural Implications of Agentic AI Workflows
- 依據 2026 年 UT Austin 與微軟 Azure 團隊首篇雲端資料中心 Agent 晶片與伺服器架構實證論文，建立算力底層視角。
- 新增概念頁：`wiki/concepts/agentic-computing-systems.md`（CPU-GPU 打乒乓球現象、三大軟體角色、微架構 Top-Down TMA 分解公式）。
- 新增實踐頁：`wiki/practices/agent-serving-and-hardware-optimization.md`（Agora 原型調優手冊、MILP 混合整數線性規劃顯存放置模型、CPU 動態收割退讓方程、任務綁核 Task Pinning）。
- 更新 `index.md`：在技術階梯圖中開闢「階梯 5：底層系統與算力架構 (Systems & Hardware Infra)」。

## [2026-11-23] ingest | GPT-3 (Few-Shot Learners) & Chain-of-Thought (CoT) Prompting
- 依據使用者審核核准，成功下載兩篇經典開山論文之 PDF，並完成結構化解析。
- 新增概念頁：`wiki/concepts/large-language-models.md`（自回歸語言建模交叉熵損失、Scaling Laws 冪律縮放、In-Context Learning 湧現）。
- 新增概念頁：`wiki/concepts/prompt-engineering.md`（思維鏈條件機率分解、Test-time Compute Scaling 物理意義、自洽性多數決）。
- 更新 `index.md`：在技術階梯圖中正式點亮「階梯 1：大語言模型」與「階梯 2：Prompt Engineering」，完成全鏈路貫通。

## [2026-11-23] synthesize | 重構與完善《AI Agent 的核心原理與實務運用》19 頁互動式 HTML 投影片
- 依據使用者審核確認之大綱與「生成 HTML 工作流圖譜（Workflow Graph）」品質門禁，重構投影片至完整 19 頁。
- 納入實際落地核心工具：
  - **The Loop（動態自主控制迴圈）**：Claude Code、Cursor 之 ReAct 與環境回饋自適應修正。
  - **The Graph（工作流圖譜/狀態機）**：Dify、Coze、LangGraph 之有向無環圖、條件分支與嚴格人工閘門（以「本次 HTML 生成流程」作為實例）。
  - **專案憲章規範**：`AGENTS.md`、`.cursorrules` 之權限防護與防幻覺鐵律。
- 修復 HTML 標籤解析錯誤（轉義 KaTeX 內部符號防止 DOM 自動巢狀吞噬），確認 19 頁 DOM 結構正常無空白。
- 檔案存放於 `presentations/ai-agent-principles-and-usage.html`。
- 連結並驗證 89 個素材圖示（`assest/` 貼紙與向量圖），支援方向鍵/空白鍵切換、全螢幕播放與頂部進度條。

## [2026-11-23] lint | 將簡報 Graph 工作流、素材庫與視覺防呆守則正式寫入 AGENTS.md
- 更新 `AGENTS.md` 知識庫目錄架構：正式收錄 `assest/` 視覺素材層與 `presentations/` 產出層。
- 新增章節 `5.2 互動式簡報生成工作流協議 (Presentation & Artifact Graph Protocol)`：
  - 定義 6 階段狀態機工作流圖譜（Mermaid 視覺化）。
  - 明訂「大綱確認」為必須等待使用者核准之強制硬門禁（Human-in-the-Loop Gate）。
  - 明訂「自動化 QA 斷言」為放行交付之自動門禁，未通過即觸發 The Loop 自主修復迴圈。
- 新增第 8 節 `8. 簡報視覺風格、素材庫與渲染防禦守則 (Presentation, Style & Asset Standards)`：
  - 定義「親和趣味圖解風（Playful & Visual）」標準（米白溫潤底色、圓角卡片、馬卡龍色標籤、雙重深度）。
  - 明訂 `assest/` 實體素材庫規範與路徑存在性斷言鐵律。
  - 明訂 KaTeX `<` / `>` 強制轉義為 `&lt;` / `&gt;` 鐵律，防杜 HTML Parser 標籤吞噬與空白頁 Bug。
