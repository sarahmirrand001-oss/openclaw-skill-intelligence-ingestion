# OpenClaw Skill: Intelligence Ingestion

> You're feeding something smarter than you.
> This Skill makes sure you know what it ate, and what it became.

**Zero-loss information ingestion pipeline** — Analyze URLs, articles, and tweets for strategic value. Classify, score, store structured notes in **Obsidian**, and update the **Strategic Landscape** capability map.

## Why This Skill?

Most people collect information but never process it. They bookmark 100 links and read 3.

This Skill flips the model: **every piece of information is analyzed, classified, scored, and mapped to your system before it enters your knowledge base.** You always know what your Agent learned and how its capabilities changed.

## Core Flow

```
READ → CLASSIFY → ANALYZE → MAP → STORE → REMEMBER → RESPOND
```

7-step pipeline. From "I saw it" to "I know what it means." Zero information loss.

## Prerequisites

| Tool | Purpose | Required? |
|------|---------|-----------|
| **Obsidian** | Knowledge storage (notes land here) | ✅ Required |
| **OpenClaw** | Skill host + Agent orchestration | ✅ Required |
| **Chrome** | For reading login-required pages (X/Twitter) | 🟡 Recommended |

## Quick Start

```bash
# 1) Install
openclaw skills install github:sarahmirrand001-oss/openclaw-skill-intelligence-ingestion

# 2) Configure
cp config.example.json config.json
# Edit config.json — set your Obsidian Vault path

# 3) Initialize Strategic Landscape (optional, auto-created on first use)
cp STRATEGIC_LANDSCAPE.template.md /path/to/your/workspace/STRATEGIC_LANDSCAPE.md

# 4) Use it — just share a link
"Analyze this: https://example.com/article"

# 5) Verify
ls /path/to/your/Obsidian_Vault/20_Intelligence/
# You should see a new note with today's date
```

## What It Produces

Each ingestion creates:
- 📄 **Structured Obsidian note** (category + strategic value score + capability change)
- 📝 **Daily memory log update**
- 🗺️ **Strategic Landscape update** (for critical information)
- � **Capability boundary assessment** (what can your Agent do now that it couldn't before?)

## File Structure

```
intelligence-ingestion/
├── SKILL.md                          # Skill behavior + 7-step pipeline
├── README.md                         # This file
├── config.example.json               # Configuration template (copy to config.json)
├── config.json                       # Your local config (gitignored)
├── STRATEGIC_LANDSCAPE.template.md   # Landscape template for first-time setup
├── index.html                        # Public landing page (deployable to Vercel)
└── vercel.json                       # Static deployment config
```

## Companion: Strategic Landscape

Intelligence Ingestion is not standalone — it works with a **Strategic Landscape** (capability map):

- **Intelligence Ingestion** = the "input" for information
- **Strategic Landscape** = the "map" of your system

Every critical piece of information automatically updates the Landscape, so you always see your Agent's full capability picture — what it can do, what's in progress, what's missing.

## X/Twitter Support

X has aggressive anti-scraping. This Skill uses a 4-level fallback chain:

1. **xurl Skill** (OpenClaw built-in) → Authenticated X API v2 access
2. **Browser tool** → Chrome login session
3. **Web search** → Search engine cache
4. **User paste** → Last resort

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Skill doesn't trigger when I share a URL | Start a new session (Skills snapshot is cached per session) |
| Obsidian note not created | Check `config.json` — is `obsidian_vault_path` correct? Does the folder exist? |
| X/Twitter link fails | Ensure `xurl` is configured, or log into X in Chrome |
| "config.json not found" | Run `cp config.example.json config.json` and edit paths |
| Strategic Landscape not updating | Check `landscape_path` in config.json matches your actual file location |

## Upgrade

```bash
# Pull latest version
openclaw skills install github:sarahmirrand001-oss/openclaw-skill-intelligence-ingestion

# Your config.json is gitignored, so it won't be overwritten
```

## Deploy Landing Page

```bash
vercel --prod
```

---

*Built for pragmatic operators. No hype, no fluff, only strategic signal.*
