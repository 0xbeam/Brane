# Brane

**Multi-source feedback-to-instructions engine for AI agents.**

Scrape Slack threads, Figma comments, tweets, and web pages — convert them into structured instruction markdown that agents can execute.

<img width="100%" alt="brane-dashboard" src="https://feedback-hub-blush.vercel.app" />

## Architecture

```
URL Input (CLI / Dashboard / API)
     ↓
 detectAdapter(url)  →  Slack | Figma | Twitter | URL
     ↓
 Browser Engine (Cloudflare → Lightpanda → fetch)
     ↓
 adapter.scrape()  →  InstructionSet
     ↓
 Categorize (blocker / revision / question / approval)
     ↓
 Generate Markdown  →  instruction.md
     ↓
 Save to output/  →  Dashboard renders results
```

### Browser Engine Fallback Chain

Brane uses a three-tier scraping strategy:

| Engine | Speed | Use Case | Config |
|--------|-------|----------|--------|
| **Cloudflare Browser Rendering** | Edge-fast | Production, JS-rendered SPAs | `CF_API_TOKEN` + `CF_ACCOUNT_ID` |
| **Lightpanda** | 11x Chrome | Local dev, self-hosted | `LIGHTPANDA_URL=ws://127.0.0.1:9222` |
| **Plain fetch** | Fastest | Static pages (default) | No config needed |

SPA domains (x.com, notion.so, linear.app, medium.com) are auto-detected and routed through browser engines. Everything else uses plain fetch.

## Quick Start

```bash
# Clone
git clone https://github.com/0xbeam/Brane.git && cd Brane

# Install
npm install

# Configure (at minimum, set your Slack token)
cp .env.example .env
# Edit .env with your tokens

# Run (API + Dashboard)
npm run dev
```

Dashboard: `http://localhost:5180`
API: `http://localhost:3210`

## Usage

### Dashboard

1. Click **"New Scrape"** in the header
2. Paste one or more URLs (Slack threads, Figma files, tweets, any web page)
3. Assign a project tag
4. Watch the live pipeline visualization as agents process each URL
5. View structured instructions with categorized feedback

### CLI

```bash
# Scrape a single URL
brane scrape https://your-slack.slack.com/archives/C.../p... -p my-project

# Dispatch multiple URLs in parallel
brane dispatch url1 url2 url3 -p my-project

# Force browser engine
brane scrape https://x.com/user/status/123 --browser

# List all scraped instructions
brane list
```

### API

```bash
# Scrape a URL
curl -X POST http://localhost:3210/api/scrape \
  -H "Content-Type: application/json" \
  -d '{"url": "https://...", "project": "my-project"}'

# Batch dispatch
curl -X POST http://localhost:3210/api/dispatch \
  -H "Content-Type: application/json" \
  -d '{"urls": ["url1", "url2"], "project": "my-project"}'

# Check job status
curl http://localhost:3210/api/jobs

# Health check (includes browser engine status)
curl http://localhost:3210/api/health
```

## Source Adapters

| Source | What It Scrapes | Auth Required |
|--------|----------------|---------------|
| **Slack** | Thread messages, reactions, images, user mentions | `SLACK_BOT_TOKEN` |
| **Figma** | File comments, threads, resolved/unresolved status | `FIGMA_TOKEN` |
| **Twitter/X** | Tweet content from URL (browser engine recommended) | None (browser helps) |
| **URL** | Any web page — text, images, metadata | None |

## Output Format

Each scrape produces:

```
output/
└── slack-1234567890-123456/
    ├── instruction.json    # Full structured data
    ├── instruction.md      # Agent-ready markdown
    └── images/             # Downloaded attachments
```

### Instruction Markdown

```markdown
# Fix the header spacing on mobile

**Source:** slack · **Project:** sanctuary
**Scraped:** Mar 17, 2026

## Context
> The header overlaps the nav on iPhone 14...

## Agent Instructions
- [ ] **[blocker]** Navigation is completely hidden on mobile — @designer
- [ ] **[revision]** Reduce header padding from 32px to 16px on <768px — @engineer
- [ ] **[question]** Should we keep the sticky behavior on mobile? — @pm

## Approvals
- ✓ @lead — "Desktop version looks great"
```

## Feedback Categories

Messages are auto-categorized using keyword + emoji signal detection:

| Category | Signal | Color |
|----------|--------|-------|
| **Blocker** | 🛑 "critical", "broken", "can't ship" | Red |
| **Revision** | ✏️ "change", "fix", "should be" | Amber |
| **Question** | ❓ "why", "how", "?" | Blue |
| **Approval** | ✅ "lgtm", "looks good", "ship it" | Green |
| **Context** | — Original post, neutral comments | Gray |

## Environment Variables

```bash
# Source adapters
SLACK_BOT_TOKEN=xoxb-...          # channels:history, files:read, users:read
FIGMA_TOKEN=figd_...               # Personal access token

# Browser engines (optional — enables JS-rendered scraping)
CF_API_TOKEN=                      # Cloudflare API token
CF_ACCOUNT_ID=                     # Cloudflare account ID
LIGHTPANDA_URL=ws://127.0.0.1:9222 # Local Lightpanda endpoint

# Server
API_PORT=3210
OUTPUT_DIR=./output
```

## Tech Stack

- **Frontend:** React 19 + Vite 7 + Tailwind 4
- **Backend:** Express 5 + Node.js
- **Design:** Gravity system (Cormorant Garamond + DM Sans + forest green #2A7A5B)
- **Browser:** Cloudflare Browser Rendering / Lightpanda / puppeteer-core
- **Deployment:** Vercel (dashboard) + any Node host (API)

## Live

Dashboard: [feedback-hub-blush.vercel.app](https://feedback-hub-blush.vercel.app)

---

Built by [Sanctuary Parc](https://github.com/0xbeam)
