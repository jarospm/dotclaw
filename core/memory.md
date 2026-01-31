# OpenClaw Memory System

How OpenClaw maintains persistent, searchable memory across sessions.

## Context vs Memory

**Context** — what the model sees per request:
- System prompt + conversation history + tool results + attachments
- Ephemeral, bounded by context window, expensive (API costs)

**Memory** — what's stored on disk:
- `MEMORY.md` + `memory/*.md` + session transcripts
- Persistent, unbounded, cheap, searchable

## Two-Layer Storage

```
~/clawd/
├── MEMORY.md              # Layer 2: Long-term curated knowledge
└── memory/
    ├── 2026-01-26.md      # Layer 1: Today's notes
    ├── 2026-01-25.md      # Yesterday's notes
    └── ...
```

**Layer 1: Daily Logs** (`memory/YYYY-MM-DD.md`)
- Append-only daily notes
- Agent writes when it wants to remember something
- Timestamped entries throughout the day

**Layer 2: Long-term Memory** (`MEMORY.md`)
- Curated, persistent knowledge
- User preferences, important decisions, key contacts
- Significant events, lessons learned

## Memory Tools

**memory_search** — semantic search across all memory files:
```json
{
  "query": "What did we decide about the API?",
  "maxResults": 6,
  "minScore": 0.35
}
```

**memory_get** — read specific content after finding it:
```json
{
  "path": "memory/2026-01-20.md",
  "from": 45,
  "lines": 15
}
```

**Writing** — no dedicated tool; uses standard `write`/`edit` tools since memory is just Markdown.

## Semantic Search Setup

Semantic search requires an **embedding provider** to convert text into vectors. Without one, `memory_search` won't work.

### Supported Providers

| Provider | Model | Notes |
|----------|-------|-------|
| **OpenAI** | `text-embedding-3-small` | Recommended. Cheap (~$0.02/1M tokens), fast, good quality |
| **Google** | Gemini embeddings | Alternative if you already have Gemini keys |
| **Local** | GGUF models via node-llama-cpp | No API costs, requires local model file |

### Adding an OpenAI Key (Recommended)

1. Get an API key from [platform.openai.com/api-keys](https://platform.openai.com/api-keys)

2. Add it to Clawdbot:
   ```bash
   clawdbot models auth paste-token --provider openai
   # Paste your key when prompted
   ```

3. Verify setup:
   ```bash
   clawdbot memory status
   ```
   Should show `Provider: openai` and `Vector: ready`

4. Index your memory files:
   ```bash
   clawdbot memory index
   ```

### Checking Status

```bash
clawdbot memory status
```

Output when working:
```
Memory Search (main)
Provider: openai (requested: openai)
Model: text-embedding-3-small
Indexed: 6/6 files · 23 chunks
Vector: ready
```

Output when missing API key:
```
No API key found for provider "openai".
```

### Troubleshooting: Batch API Issues

OpenAI's Batch API (used by default for bulk indexing) can get stuck on some accounts. If indexing hangs, disable batch mode:

```bash
# In clawdbot.json, add:
{
  "agents": {
    "defaults": {
      "memorySearch": {
        "provider": "openai",
        "model": "text-embedding-3-small",
        "remote": {
          "batch": {
            "enabled": false
          }
        }
      }
    }
  }
}
```

Then restart the gateway and re-run `clawdbot memory index`.

## How Indexing Works

1. File saved → file watcher detects change (1.5s debounce)
2. **Chunking** — split into ~400 token chunks with 80 token overlap
3. **Embedding** — each chunk → embedding provider → vector (1536 dimensions)
4. **Storage** — SQLite with sqlite-vec (vectors) + FTS5 (full-text)

Index location: `~/.clawdbot/memory/<agentId>.sqlite`

## How Search Works

Two strategies run in parallel:
- **Vector search** (semantic) — finds content that means the same thing
- **BM25 search** (keyword) — finds content with exact tokens

Combined scoring: `finalScore = (0.7 × vectorScore) + (0.3 × textScore)`

Results below `minScore` threshold (default 0.35) are filtered out.

## Compaction

When context approaches the limit, older conversation is summarized:

1. Summarize old turns into a compact summary
2. Keep recent turns intact
3. Persist summary to JSONL transcript

**Memory Flush** — before compaction, agent gets a silent turn to write important info to memory files, preventing data loss.

## Pruning

Trims old tool outputs (which can be huge) without rewriting history:

- **Soft trim** — keep head + tail of large outputs
- **Hard clear** — replace with placeholder text

**Cache-TTL Pruning** — detects when Anthropic's prompt cache expires and trims before next request to reduce re-caching cost.

## Multi-Agent Isolation

Each agent has separate:
- Workspace (`~/clawd/`, `~/clawd-work/`, etc.)
- SQLite index (`~/.clawdbot/memory/<agentId>.sqlite`)

No cross-agent memory search by default.

## Key Principles

- **Transparency** — memory is plain Markdown, editable and version-controllable
- **Search over injection** — agent searches for relevant info vs stuffing context
- **Persistence** — important info survives in files, not just conversation
- **Hybrid search** — vectors for semantics + keywords for exact matches

---

Source: [Manthan Gupta's blog post](https://manthanguptaa.in/) via [@manthanguptaa](https://x.com/manthanguptaa/status/2015780646770323543)
