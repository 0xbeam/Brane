# brane

**Multi-source feedback aggregation engine. Slack threads, Figma comments, tweets, web pages — scraped and converted into structured agent instructions.**

[![Node.js](https://img.shields.io/badge/Node.js-22+-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Express](https://img.shields.io/badge/Express-5-000000?logo=express)](https://expressjs.com)
[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)](https://react.dev)
[![Slack API](https://img.shields.io/badge/Slack_API-4A154B?logo=slack&logoColor=white)](https://api.slack.com)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## What it does

Paste any URL. Brane detects the source, scrapes it through the right adapter, categorizes every piece of feedback, and outputs agent-ready instruction markdown.

```
URL → detectAdapter() → Browser Engine → adapter.scrape() → Categorize → instruction.md
```

## Features

| | Feature | Details |
|---|---|---|
| **Adapters** | Slack, Figma, Twitter/X, generic URL | Auto-detected from URL pattern |
| **Browser fallback** | Cloudflare → Lightpanda → fetch | Three-tier chain, SPA-aware routing |
| **Categorization** | Blocker / Revision / Question / Approval | Keyword + emoji signal detection |
| **Output** | Structured JSON + agent markdown | Checklist format with severity tags |
| **Dashboard** | React 19 + Vite 7 | Live pipeline visualization |
| **CLI** | `brane scrape`, `brane dispatch`, `brane list` | Parallel dispatch, browser flags |
| **API** | REST endpoints | Scrape, batch dispatch, job status |

## Quick start

```bash
git clone https://github.com/0xbeam/brane.git && cd brane
npm install
cp .env.example .env   # add your tokens
npm run dev             # API :3210 + Dashboard :5180
```

## CLI

| Command | Description |
|---|---|
| `brane scrape <url> -p <project>` | Scrape a single URL |
| `brane dispatch <url1> <url2> -p <project>` | Parallel multi-URL scrape |
| `brane scrape <url> --browser` | Force browser engine |
| `brane list` | List all scraped instructions |

## API

```bash
# Scrape
curl -X POST :3210/api/scrape -H "Content-Type: application/json" \
  -d '{"url": "https://...", "project": "my-project"}'

# Batch dispatch
curl -X POST :3210/api/dispatch -H "Content-Type: application/json" \
  -d '{"urls": ["url1", "url2"], "project": "my-project"}'

# Job status
curl :3210/api/jobs

# Health (includes browser engine status)
curl :3210/api/health
```

## Architecture

```
┌─────────────────────────────────────────────────┐
│  Input: CLI / Dashboard / API                   │
└──────────────────┬──────────────────────────────┘
                   ▼
          detectAdapter(url)
           ┌───┬───┬───┬───┐
           │ S │ F │ T │ U │  Slack · Figma · Twitter · URL
           └─┬─┴─┬─┴─┬─┴─┬─┘
             └───┴───┴───┘
                   ▼
        Browser Engine (fallback chain)
        ┌────────────────────────┐
        │ 1. Cloudflare Rendering│  edge-fast, JS SPAs
        │ 2. Lightpanda          │  11x Chrome, local dev
        │ 3. Plain fetch         │  static pages, default
        └────────┬───────────────┘
                 ▼
         adapter.scrape() → InstructionSet
                 ▼
         Categorize (keyword + emoji signals)
                 ▼
         Generate markdown → output/<id>/
           ├── instruction.json
           ├── instruction.md
           └── images/
```

### Adapters

| Source | Scrapes | Auth |
|---|---|---|
| **Slack** | Thread messages, reactions, images, user mentions | `SLACK_BOT_TOKEN` |
| **Figma** | File comments, threads, resolved status | `FIGMA_TOKEN` |
| **Twitter/X** | Tweet content from URL | None (browser recommended) |
| **URL** | Any web page — text, images, metadata | None |

### Feedback categories

| Category | Signal | Tag |
|---|---|---|
| Blocker | "critical", "broken", "can't ship" | `[blocker]` |
| Revision | "change", "fix", "should be" | `[revision]` |
| Question | "why", "how", "?" | `[question]` |
| Approval | "lgtm", "looks good", "ship it" | `[approval]` |

## Configuration

```bash
# Source adapters
SLACK_BOT_TOKEN=xoxb-...            # channels:history, files:read, users:read
FIGMA_TOKEN=figd_...                 # Personal access token

# Browser engines (optional)
CF_API_TOKEN=                        # Cloudflare API token
CF_ACCOUNT_ID=                       # Cloudflare account ID
LIGHTPANDA_URL=ws://127.0.0.1:9222   # Local Lightpanda endpoint

# Server
API_PORT=3210
OUTPUT_DIR=./output
```

## Stack

React 19 · Vite 7 · Tailwind 4 · Express 5 · Node.js · Cloudflare Browser Rendering · Lightpanda · puppeteer-core

## Live

[feedback-hub-blush.vercel.app](https://feedback-hub-blush.vercel.app)

---

Built by [0xbeam](https://github.com/0xbeam)
