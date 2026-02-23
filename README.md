# arXiv 每日論文分析自動化流水線

每日自動抓取 arXiv 最新機器人學與電腦視覺論文，透過 Gemini AI 產出深度分析筆記，並自動發布至 HackMD。

> 本專案完全透過 **Vibe Coding** 開發——從工作流設計、除錯到部署，全程以 Claude Code 作為 AI 驅動的開發夥伴。

**[作品集完整導覽（含開發過程與設計思考）](https://hackmd.io/@tYriCP5OSzuVXnDOe5UwOA/BJ5HA6YuZg)**

## 系統架構

```
┌─────────────┐     ┌──────────────┐     ┌────────────────┐
│  每日排程     │────▶│  arXiv API   │────▶│  XML 解析器     │
│  (24 小時)   │     │  (HTTP GET)  │     │  (JavaScript)  │
└─────────────┘     └──────────────┘     └───────┬────────┘
                                                  │ 最新 3 篇論文
                                                  ▼
┌─────────────┐     ┌──────────────┐     ┌────────────────┐
│  HackMD     │◀────│  Markdown    │◀────│  Gemini 2.5    │
│  (自動發布)  │     │  格式整合     │     │  Pro 深度分析   │
└─────────────┘     └──────────────┘     └────────────────┘
```

## 功能特色

### 智慧論文抓取
- 搜尋 **cs.RO**（機器人學）與 **cs.CV**（電腦視覺）分類
- 關鍵字：`robotic arm`、`robot manipulation`、`machine vision`、`visual perception`、`motion planning`、`grasp planning`、`visual servoing`
- **日期感知過濾**：優先取最近 3 天內的論文，不足則擴大到 7 天——避免重複分析

### AI 深度分析（13 個段落）
每篇論文由 Gemini 2.5 Pro 以教授級 prompt 產出：

| 段落 | 說明 |
|------|------|
| 一句話總結 | 30 字內精準概括核心貢獻 |
| 論文摘要（中文） | 400 字完整中文摘要 |
| 研究背景與動機 | 領域現況、痛點分析、切入角度 |
| 核心貢獻與創新點 | 條列式詳細說明 |
| 方法論詳解 | 完整 pipeline、模組拆解、損失函數、方法對比表 |
| 實驗結果與分析 | 實驗設置、量化結果、消融實驗、結果解讀 |
| 心智圖 | 互動式 Markmap（HackMD 原生支援） |
| 簡報大綱 | 10 張投影片含講者備註 |
| 關鍵詞彙表 | 10+ 術語含本論文具體意義 |
| 論文評析 | 優點、限制、審稿人角度改進建議 |
| 與機械手臂/機器視覺的關聯性 | 應用場景、啟發、整合考量 |
| 實作可行性 | 資源需求、開源碼、難度評估 |
| 延伸閱讀 | 5 篇相關論文含關係標註 |

### 精美排版輸出
- 自訂字體：**LXGW WenKai TC**（中文）+ **Times New Roman**（英文）
- 互動式心智圖（Markmap）
- 自動產生目錄
- 發布至 HackMD，僅限擁有者存取

## 技術棧

| 元件 | 技術 |
|------|------|
| **流程編排** | [n8n](https://n8n.io/)（自架於 HuggingFace Spaces） |
| **論文來源** | [arXiv API](https://info.arxiv.org/help/api/basics.html) |
| **AI 分析** | Gemini 2.5 Pro（透過 OpenAI 相容代理） |
| **LLM 代理** | [CLIProxyAPI](https://github.com/nicepkg/CLIProxyAPI)（部署於 HuggingFace Spaces） |
| **筆記發布** | [HackMD API](https://hackmd.io/@hackmd-api/developer-portal) |
| **開發方式** | Claude Code（Vibe Coding） |

## 安裝指南

### 前置需求
- n8n 實例（自架或雲端版）
- Gemini API 存取（API Key 或透過 CLIProxyAPI 的 OAuth）
- HackMD API Token

### 1. 匯入工作流
將 `workflow.json` 匯入你的 n8n 實例：**Settings > Import Workflow**

### 2. 設定 API 金鑰
替換工作流中的佔位符：

| 佔位符 | 說明 |
|--------|------|
| `YOUR_LLM_API_ENDPOINT` | Gemini API 端點（如 `https://generativelanguage.googleapis.com` 或代理位址） |
| `YOUR_API_KEY` | Gemini API Key 或代理 API Key |
| `YOUR_HACKMD_API_TOKEN` | HackMD API Token（從 [hackmd.io/settings](https://hackmd.io/settings) 取得） |

### 3. 自訂搜尋條件
編輯「抓取 arXiv」節點中的查詢 URL：
- 修改 `cat:cs.RO+OR+cat:cs.CV` 以搜尋不同分類
- 調整 `all:` 欄位中的關鍵字
- 調整 `max_results` 控制候選論文數量

### 4. 啟用
將工作流切換為 **Active**——每 24 小時自動執行一次。

## 速率限制處理

工作流內建 Gemini API 速率限制控制：
- **批次大小**：每次 1 個請求
- **批次間隔**：每個請求間隔 65 秒
- **逾時設定**：每個請求 180 秒

確保符合免費版額度限制（Gemini CLI OAuth 約 2 RPM）。

## 範例輸出

每日執行會產出一份 HackMD 筆記：

```markdown
# arXiv 每日論文筆記 - 2026-02-23

## 目錄
1. 論文標題 1
2. 論文標題 2
3. 論文標題 3

---

# 論文 1: 論文標題 1
| 項目 | 內容 |
|---|---|
| **連結** | http://arxiv.org/abs/xxxx |
| **作者** | 作者1, 作者2 |

## 🎯 一句話總結
...

## 📝 論文摘要（中文）
...

## 🔧 方法論詳解
...

## 🧠 心智圖 (Markmap)
...

（每篇論文包含 13 個詳細分析段落）
```

## 專案結構

```
arxiv-daily-pipeline/
├── README.md              # 本文件
├── workflow.json           # n8n 工作流定義（已移除 API 金鑰）
├── docs/
│   └── prompt.md          # 完整 Gemini 分析 prompt
├── examples/
│   └── sample-output.md   # HackMD 輸出範例
└── LICENSE                # MIT 授權
```

## 開發歷程

本專案完全透過 **Vibe Coding** 搭配 Claude Code 開發：

1. **基礎設施部署** — 在 HuggingFace Spaces 上以 Docker 部署 n8n 與 CLIProxyAPI
2. **工作流設計** — 透過 AI 對話迭代設計 6 節點的 n8n 流水線
3. **Prompt 工程** — 從基礎摘要 prompt 逐步演進為 13 段落的教授級深度分析
4. **除錯優化** — 解決 arXiv API 編碼問題、Gemini 429 速率限制、模型可用性、HF Spaces 部署挑戰
5. **細節打磨** — 加入自訂字體（LXGW WenKai TC）、日期過濾、批次間隔最佳化

## 授權

MIT
