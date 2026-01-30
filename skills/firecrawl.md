# Firecrawl — Web Research, Search, Scrape, Map

Comprehensive web tool for search, scraping, mapping, and research. Replaces built-in web_search and enhances web_fetch.

---

## Skill Type

**Custom** — installed separately via `npx skills add firecrawl/cli`.

| Component | Location |
|-----------|----------|
| Skill (SKILL.md) | `~/clawd/.agents/skills/firecrawl/SKILL.md` |
| CLI binary | `~/.npm-global/bin/firecrawl` |
| CLI credentials | `~/.config/firecrawl-cli/` |
| Gateway credentials | `~/.config/systemd/user/clawdbot-gateway.service.d/firecrawl.conf` |

---

## What It Does

- **Search** — Web, news, image search with optional scraping
- **Scrape** — Extract content from any URL (handles JS, anti-bot)
- **Map** — Discover all URLs on a website
- **Crawl** — Scrape entire sites

**Why use over built-in tools:**
- Handles JavaScript-rendered content
- Bypasses anti-bot protections
- Returns LLM-optimized markdown
- Search + scrape in one command
- No separate Brave API key needed

---

## Installation

**CLI:**
```bash
npm install -g firecrawl-cli
```

**Skill (for agents):**
```bash
npx skills add firecrawl/cli -y
```

Installed to: `~/clawd/.agents/skills/firecrawl/`

**Verify:**
```bash
firecrawl --status
```

---

## Authentication

### API Key

Get from: https://firecrawl.dev (has free tier)

### Two places credentials can live:

**1. Firecrawl's own config** (for CLI direct use):
```bash
firecrawl login --api-key fc-YOUR-KEY
# Stored in: ~/.config/firecrawl-cli/
```

**2. Systemd drop-in** (for Clawdbot's web_fetch fallback):
```
~/.config/systemd/user/clawdbot-gateway.service.d/firecrawl.conf
```

Both can coexist — CLI uses its own config, Clawdbot uses the env var.

---

## Common Commands

### Search

```bash
# Basic search
firecrawl search "your query" --limit 10

# Search + scrape results
firecrawl search "AI agents" --scrape -o .firecrawl/search-ai.json --json

# News search
firecrawl search "tech news" --sources news --tbs qdr:d  # past day

# Image search
firecrawl search "landscapes" --sources images

# Filter by category
firecrawl search "python tutorial" --categories github
```

### Scrape

```bash
# Basic scrape
firecrawl scrape https://example.com -o .firecrawl/example.md

# Main content only (no nav/footer)
firecrawl scrape https://example.com --only-main-content

# Wait for JS to render
firecrawl scrape https://spa-site.com --wait-for 3000

# Multiple formats
firecrawl scrape https://example.com --format markdown,links -o .firecrawl/example.json
```

### Map

```bash
# All URLs on a site
firecrawl map https://example.com -o .firecrawl/urls.txt

# Filter by keyword
firecrawl map https://example.com --search "blog"

# Include subdomains
firecrawl map https://example.com --include-subdomains
```

---

## File Organization

The skill creates a `.firecrawl/` folder for results:

```
~/clawd/.firecrawl/
├── search-query.json
├── example.com.md
├── urls.txt
└── scratchpad/     # Temporary scripts
```

Add to `.gitignore`:
```
.firecrawl/
```

---

## Skill Location

```
~/clawd/.agents/skills/firecrawl/
├── SKILL.md           # Full documentation
└── rules/
    └── install.md     # Auth instructions
```

---

## How Agent Uses It

The skill tells the agent to:
1. Prefer `firecrawl` over `web_fetch` and `web_search`
2. Save output to files (not flood context)
3. Use grep/head to read results incrementally
4. Run multiple scrapes in parallel

---

## Parallel Scraping

```bash
# Run multiple scrapes at once
firecrawl scrape https://site1.com -o .firecrawl/1.md &
firecrawl scrape https://site2.com -o .firecrawl/2.md &
firecrawl scrape https://site3.com -o .firecrawl/3.md &
wait
```

Check concurrency limit with `firecrawl --status`

---

## Troubleshooting

**"Not authenticated":**
```bash
firecrawl login --api-key fc-YOUR-KEY
```

**Check credits:**
```bash
firecrawl --status
```

**Rate limited:**
- Check concurrency limit in status
- Reduce parallel jobs

---

## References

- Service: https://firecrawl.dev
- Docs: https://docs.firecrawl.dev
- CLI Docs: https://docs.firecrawl.dev/sdks/cli
- Skill: `~/clawd/.agents/skills/firecrawl/SKILL.md`
