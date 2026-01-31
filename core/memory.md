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
