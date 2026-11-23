# Model Context Protocol (MCP): Landscape, Security Threats, and Future Research Directions

- **作者**：Xinyi Hou, Yanjie Zhao, Shenao Wang, Haoyu Wang (Huazhong University of Science and Technology)
- **發表時間**：April 2025 (ACM Reference Format)
- **文獻性質**：首篇針對 MCP（Model Context Protocol）生態架構、發展現狀與安全威脅的學術綜述與 Vision Paper。
- **原始來源**：`raw/papers/mcp-landscape-security-2025.md`

---

## 報告核心內容精煉

### 1. 誕生背景與演進痛點 (Background & Motivation)
- **演進歷史**：
  1. **手動 API 串接 (Manual API Wiring)**：開發者為每個服務客製化身份驗證、資料轉換與錯誤處理，維護成本巨大且緊密耦合。
  2. **外掛介面 (ChatGPT Plugins / Coze / Yuanqi, 2023)**：基於 OpenAPI 規範，但多為單向呼叫，無法維護跨步驟狀態，且形成各平台封閉的孤島。
  3. **Agent 框架工具呼叫 (LangChain, LlamaIndex)**：雖有封裝，但依賴預先定義工作流，缺乏自主發現與跨平台互通標準。
  4. **RAG (檢索增強生成)**：僅限於「被動讀取」，無法主動修改外部狀態或觸發工作流。
- **MCP 的誕生**：Anthropic 於 2024 年底推出，受微軟 Language Server Protocol (LSP) 啟發，旨在建立一個「開放、平台中立、動態發現」的通用協議，成為 AI 模型與外部工具、資源互動的標準介面。

### 2. MCP 核心架構三要素 (Core Components)
- **MCP Host**：執行 AI 任務與客戶端的宿主環境（如 Claude Desktop、Cursor、Windsurf、自主 AI 代理）。
- **MCP Client**：宿主環境內的通訊中介，負責向 MCP Server 查詢功能清單、發起呼叫、處理通知與取樣（Sampling）。
- **MCP Server**：向 Host/Client 提供外部系統存取能力的服務端，提供三大核心能力：
  1. **Tools（工具）**：可執行的操作（如呼叫外部 API、執行 Bash、讀寫資料庫）。
  2. **Resources（資源）**：向模型暴露的結構化/非結構化資料源（如本地文件、即時日誌、雲端檔案）。
  3. **Prompts（提示詞範本）**：預先配置的工作流與提示詞範本。

### 3. 生態現狀與主流採納者 (Landscape)
- **核心推手與採納者**：
  - 模型與框架：Anthropic (Claude)、OpenAI (Agent SDK / Responses API)、Baidu Maps。
  - 開發者工具與 IDE：Cursor (Composer 整合)、Windsurf、Zed、JetBrains、Cline、Replit。
  - 雲端與基礎設施：Cloudflare (託管遠端 Remote MCP 與 OAuth)、Stripe (支付 API MCP 化)、Block/Square。
- **社群註冊庫與工具**：
  - 社群註冊庫：MCP.so (4700+ servers)、Glama、PulseMCP、Smithery。
  - 自動安裝工具：Smithery CLI、mcp-get、mcp-installer。

### 4. MCP 生命週期與關鍵安全威脅 (Security & Lifecycle)
- **建立期 (Creation Phase)**：
  - *名稱混淆/搶注 (Name Collision)*：攻擊者註冊 `mcp-github` 模仿官方 `github-mcp` 攔截敏感資訊。
  - *安裝器詐欺 (Installer Spoofing)*：第三方一鍵安裝腳本夾帶後門。
  - *代碼注入/後門 (Code Injection)*。
- **運行期 (Operation Phase)**：
  - *工具名稱與語義衝突 (Tool Name Conflict)*：惡意工具在描述中注入提示詞「優先使用本工具 (prioritize this tool)」，達成「工具流劫持 (Toolflow Hijacking)」。
  - *斜線指令衝突 (Slash Command Overlap)*。
  - *沙盒逃逸 (Sandbox Escape)*：逃逸出限制容器取得宿主機 root 權限。
- **更新期 (Update Phase)**：
  - *權限殘留 (Privilege Persistence)*：API Key 撤銷未同步。
  - *漏洞版本回滾與配置漂移 (Configuration Drift)*。
