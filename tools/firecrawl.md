# Firecrawl — Web Fetch Fallback

Hosted service for extracting content from JS-heavy and anti-bot protected websites.

---

## What It Does

- Extracts readable content from web pages
- Handles JavaScript-rendered content
- Bypasses basic anti-bot protections
- Returns clean markdown/text

**Used as:** Fallback for `web_fetch` when local Readability extraction fails.

---

## How It Works

```
web_fetch request
        ↓
   Readability (local)
        ↓
   [Success?] → Return content
        ↓ [Fail]
   Firecrawl API
        ↓
   Return content
```

Firecrawl runs headless browsers in the cloud to render pages fully before extracting content.

---

## Installation

**No CLI to install** — Firecrawl is a cloud service accessed via API.

The `web_fetch` tool in Clawdbot automatically uses it when:
1. API key is configured
2. Local Readability extraction fails

---

## Configuration

### API Key

Get from: https://firecrawl.dev (has free tier)

### Where Credentials Live

Systemd drop-in:
```
~/.config/systemd/user/clawdbot-gateway.service.d/firecrawl.conf
```

Contents:
```ini
[Service]
Environment="FIRECRAWL_API_KEY=fc-your-key-here"
```

### Updating Credentials

```bash
# Edit the file
nano ~/.config/systemd/user/clawdbot-gateway.service.d/firecrawl.conf

# Reload and restart
systemctl --user daemon-reload
systemctl --user restart clawdbot-gateway
```

---

## Usage

You don't call Firecrawl directly. It's used automatically by `web_fetch`:

```
Agent: web_fetch("https://some-js-heavy-site.com")
        ↓
Clawdbot: Tries Readability → fails
        ↓
Clawdbot: Tries Firecrawl → succeeds
        ↓
Agent: Gets clean markdown content
```

---

## When Firecrawl Helps

| Site Type | Readability | Firecrawl |
|-----------|-------------|-----------|
| Static HTML blogs | ✅ Works | Not needed |
| JS-rendered SPAs | ❌ Fails | ✅ Works |
| Anti-bot protected | ❌ Fails | ✅ Usually works |
| Login required | ❌ Fails | ❌ Fails (use browser) |

---

## Clawdbot Config (Optional)

Can also configure in `~/.clawdbot/clawdbot.json`:

```json
{
  "tools": {
    "web": {
      "fetch": {
        "firecrawl": {
          "enabled": true,
          "apiKey": "fc-your-key-here",
          "onlyMainContent": true,
          "timeoutSeconds": 60
        }
      }
    }
  }
}
```

But systemd drop-in is preferred (keeps secrets out of config file).

---

## Troubleshooting

**Firecrawl not being used:**
```bash
# Check if env var is loaded
systemctl --user cat clawdbot-gateway.service | grep FIRECRAWL
```

**Still failing on some sites:**
- Some sites block even Firecrawl
- Use the browser tool for these: `clawdbot browser open <url>`

---

## References

- Service: https://firecrawl.dev
- Docs: https://docs.firecrawl.dev
- Pricing: Free tier available, paid for higher volume
