# arXiv Daily Paper Analysis Pipeline

Automated daily pipeline that fetches the latest robotics and computer vision papers from arXiv, generates in-depth analysis using Gemini AI, and publishes structured study notes to HackMD.

> Built entirely through **Vibe Coding** — the entire workflow was designed, debugged, and deployed using Claude Code as the AI-driven development partner.

## Architecture

```
┌─────────────┐     ┌──────────────┐     ┌────────────────┐
│  Schedule    │────▶│  arXiv API   │────▶│  XML Parser    │
│  (24h)      │     │  (HTTP GET)  │     │  (JavaScript)  │
└─────────────┘     └──────────────┘     └───────┬────────┘
                                                  │ Top 3 papers
                                                  ▼
┌─────────────┐     ┌──────────────┐     ┌────────────────┐
│  HackMD     │◀────│  Markdown    │◀────│  Gemini 2.5    │
│  (Publish)  │     │  Formatter   │     │  Pro Analysis  │
└─────────────┘     └──────────────┘     └────────────────┘
```

## Features

### Intelligent Paper Fetching
- Searches **cs.RO** (Robotics) and **cs.CV** (Computer Vision) categories
- Keywords: `robotic arm`, `robot manipulation`, `machine vision`, `visual perception`, `motion planning`, `grasp planning`, `visual servoing`
- **Date-aware filtering**: Prioritizes papers from the last 3 days, falls back to 7 days — no duplicate analysis

### AI-Powered Deep Analysis (13 Sections)
Each paper is analyzed by Gemini 2.5 Pro with a professor-level prompt producing:

| Section | Description |
|---------|-------------|
| One-line Summary | Core contribution in under 30 characters |
| Abstract (Chinese) | 400-word comprehensive summary |
| Background & Motivation | Field status, pain points, novel angle |
| Core Contributions | Detailed breakdown with significance |
| Methodology | Full pipeline, modules, loss functions, comparison tables |
| Experiments | Setup, results, ablation studies, interpretation |
| Mind Map | Interactive Markmap (HackMD native support) |
| Presentation Outline | 10 slides with speaker notes |
| Vocabulary Table | 10+ key terms with contextual explanations |
| Critical Analysis | Strengths, limitations, reviewer suggestions |
| Robotics Relevance | Applications to robotic arm / machine vision |
| Implementation Feasibility | Resources, open source, difficulty rating |
| Further Reading | 5 related papers with relationship tags |

### Beautiful Output
- Custom fonts: **LXGW WenKai TC** (Chinese) + **Times New Roman** (English)
- Interactive mind maps via Markmap
- Auto-generated table of contents
- Published to HackMD with owner-only permissions

## Tech Stack

| Component | Technology |
|-----------|-----------|
| **Orchestration** | [n8n](https://n8n.io/) (self-hosted on HuggingFace Spaces) |
| **Paper Source** | [arXiv API](https://info.arxiv.org/help/api/basics.html) |
| **AI Analysis** | Gemini 2.5 Pro via OpenAI-compatible proxy |
| **LLM Proxy** | [CLIProxyAPI](https://github.com/nicepkg/CLIProxyAPI) on HuggingFace Spaces |
| **Publishing** | [HackMD API](https://hackmd.io/@hackmd-api/developer-portal) |
| **Development** | Claude Code (Vibe Coding) |

## Setup Guide

### Prerequisites
- n8n instance (self-hosted or cloud)
- Gemini API access (API key or OAuth via CLIProxyAPI)
- HackMD API token

### 1. Import Workflow
Import `workflow.json` into your n8n instance via **Settings > Import Workflow**.

### 2. Configure API Keys
Replace the placeholder values in the workflow:

| Placeholder | Description |
|------------|-------------|
| `YOUR_LLM_API_ENDPOINT` | Your Gemini API endpoint (e.g., `https://generativelanguage.googleapis.com` or a proxy) |
| `YOUR_API_KEY` | Your Gemini API key or proxy API key |
| `YOUR_HACKMD_API_TOKEN` | HackMD API token from [hackmd.io/settings](https://hackmd.io/settings) |

### 3. Customize Search
Edit the arXiv query URL in the "抓取 arXiv" node to match your research interests:
- Change `cat:cs.RO+OR+cat:cs.CV` for different categories
- Modify the keyword list in the `all:` fields
- Adjust `max_results` for more/fewer candidates

### 4. Activate
Toggle the workflow to **Active** — it will run every 24 hours automatically.

## Rate Limiting

The workflow includes built-in rate limiting for Gemini API:
- **Batch size**: 1 request at a time
- **Batch interval**: 65 seconds between requests
- **Timeout**: 180 seconds per request

This ensures compliance with free-tier limits (~2 RPM for Gemini CLI OAuth).

## Sample Output

Each daily run produces a HackMD note like:

```markdown
# arXiv 每日論文筆記 - 2026-02-23

## 目錄
1. Paper Title 1
2. Paper Title 2
3. Paper Title 3

---

# 論文 1: Paper Title 1
| 項目 | 內容 |
|---|---|
| **連結** | http://arxiv.org/abs/xxxx |
| **作者** | Author1, Author2 |

## 🎯 一句話總結
...

## 📝 論文摘要（中文）
...

## 🔧 方法論詳解
...

## 🧠 心智圖 (Markmap)
...

(13 detailed analysis sections per paper)
```

## Project Structure

```
arxiv-daily-pipeline/
├── README.md              # This file
├── workflow.json           # n8n workflow definition (API keys removed)
├── docs/
│   └── prompt.md          # Full Gemini analysis prompt
├── examples/
│   └── sample-output.md   # Example HackMD output
└── LICENSE                # MIT License
```

## Development Story

This entire project was built through **Vibe Coding** with Claude Code:

1. **Infrastructure Setup** — Deployed n8n and CLIProxyAPI on HuggingFace Spaces using Docker
2. **Workflow Design** — Iteratively designed the 6-node n8n pipeline through AI-driven conversation
3. **Prompt Engineering** — Evolved from a basic summary prompt to a 13-section professor-level analysis
4. **Debugging** — Resolved arXiv API encoding issues, Gemini 429 rate limiting, model availability, and HF Spaces deployment challenges
5. **Polish** — Added custom fonts (LXGW WenKai TC), date filtering, batch interval optimization

## License

MIT
