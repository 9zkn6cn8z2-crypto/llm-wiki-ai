---
title: GitHub Copilot
aliases: [Copilot, GitHub Copilot Workspace, Copilot Agent Mode]
type: entity
tags: [ai, entity, tool, microsoft, github, coding-agent]
created: 2026-11-23
updated: 2026-11-23
status: evolving
sources:
  - "[[raw/papers/microsoft-ai-diffusion-q1-2026.md]]"
---

# GitHub Copilot

## 概述
**GitHub Copilot** 是由 GitHub 與微軟（串接 OpenAI 及後續多模型生態）推出的 AI 輔助開發工具。它見證並引領了整個 AI 軟體開發的範式轉移——從最初的「IDE 內單行代碼補全」，演進為橫跨編輯器、終端命令列、專案管理與自動發起 Pull Request 的**全功能 AI 程式開發代理平台（Agentic Platform）**。

---

## 技術規格與關鍵特性演進 (基準日: 2026 初)

### 1. 多模型靈活切換 (Multi-Model Support)
不再局限於單一模型，開發者可依據任務複雜度與成本在主流 SOTA 模型間切換：
- 支援 OpenAI 系列（如 GPT-4o、[[OpenAI Codex]] / GPT-5 系列）。
- 支援 Anthropic 系列（如 Claude 3.5 Sonnet、[[Claude Opus 4.5]]）。
- 支援 Google Gemini 系列。

### 2. 從「單行補全」到「Agent 自主行動」
- **歷史階段 1（Copilot In-line Suggestion）**：僅在程式碼編輯器內根據前後文預測下一行或下一個函式。
- **歷史階段 2（Copilot Chat & Workspace）**：加入對話視窗與全專案脈絡查詢。
- **歷史階段 3（Copilot Agent Mode）**：
  - **終端機整合（CLI Expansion）**：能直接在命令列環境執行測試、讀取編譯報錯並自我修復。
  - **自主 PR 提交（Agentic PRs）**：從指派一個 GitHub Issue 開始，Copilot Agent 可自行在雲端分支編寫代碼、通過測試並建立完整的 Pull Request 供工程師 Review。

---

## 相關實踐與概念
- 核心實踐：`[[AI 輔助軟體工程實踐]]`
- 相關概念：`[[AI 程式碼代理人]]`, `[[Vibe Coding]]`, `[[Tool Use]]`
- 競爭與替代工具：Cursor, Windsurf, Claude Code
