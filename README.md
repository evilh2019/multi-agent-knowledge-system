# Multi-Agent Knowledge System v2

> A four-layer retrieval architecture for 5 AI agents sharing one knowledge base across 3 platforms.

**Problem:** 5 AI agents on different platforms (Hermes, OpenClaw, StepCode) each have independent memory, but need to share the same knowledge. Traditional approaches either duplicate knowledge (drift) or force a single database (platform incompatibility).

**Solution:** Four-layer progressive retrieval (L3→L2→L1→L0) with a unified governance layer. Every agent queries in the same order, stopping on first hit. Misses are queued as "demands" for the Editor Agent to resolve. All operations write to an append-only log.

**Scale:** 5 agents, 198 active skills, 430+ vault files, 446 knowledge base entries, 20 cron jobs.

---

## Architecture

```
┌─────────────────────────────────────────────┐
│ L3  Hindsight — Long-term semantic memory    │
│     Fastest retrieval, cross-session         │
├─────────────────────────────────────────────┤
│ L2  Obsidian Vault — Compiled knowledge      │
│     Formal notes, source cards, taxonomy     │
├─────────────────────────────────────────────┤
│ L2.5 Project repos — STATE.md, CHANGELOG     │
│     Trust: 0.8 (project owner)               │
├─────────────────────────────────────────────┤
│ L1  IMA — Raw knowledge base (Feishu)        │
│     PDFs, articles, notes — 446 entries      │
├─────────────────────────────────────────────┤
│ L0  External search — Web (Editor only)      │
│     Hit → write IMA source card → return     │
└─────────────────────────────────────────────┘
```

**Retrieval rule:** L3→L2→L2.5→L1→L0. Stop on first hit. All missed → write demand file.

**Design rationale:** L3 first (speed — semantic search over recent conversations), L2 second (authority — formal compiled knowledge), L1 third (coverage — raw archive), L0 last (freshness — web). This is "speed → authority → coverage → freshness."

---

## Agent Roles

```
          Thinker              Executor
Personal  mama1 (Hermes)  ←→  xiexie1 (OpenClaw)
Work      oscarma1 (Hermes) ←→ oscarxia1 (OpenClaw)
Solo                           cc1 (StepCode)
```

| Agent | Tier | Role | Vault Write | IMA |
|-------|:----:|------|:-----------:|:---:|
| mama1 | **L3 Editor** | Full governance, architecture | Full | Primary |
| OscarMa1 | L2 Domain Writer | UX + AI domain | Scoped | UX+AI |
| XieXie1 | L1 Proposal | Proposals + source cards | Scoped | Self |
| OscarXia1 | L1 Proposal | Work domain proposals | Scoped | Self |
| CC1 | L0 Read-only | Query + write demands | None | None |

**Key constraint:** The Editor Agent is the sole full-writer. Other agents write only in scoped domains. This prevents knowledge drift from multiple agents compiling the same concept differently.

---

## Data Flow

```
User Input
    │
    ├──→ Knowledge Reflex (query → retrieve → answer)
    │      L3→L2→L2.5→L1→L0
    │      Hit → answer
    │      Miss → write demand
    │
    ├──→ Active Build (sync → compile)
    │      IMA new → Source Card → Compiled Note → Hindsight
    │
    └──→ activity-log (append-only)
           │
           kb-notify (L4 notification layer)
           Every 5min: tail log → agent-chat push
```

Two paths:
- **Reflex** (passive): User asks → agent retrieves → answers. The common path.
- **Build** (active): New material → extract source card → compile → persist to Hindsight. Editor-triggered.

---

## Governance

### Activity Log (append-only)

All agent operations append to `_meta/agent-activity-log.md`:

```
| date | agent_id | action | target | status | notes |
```

**Append-only is a hard constraint.** Overwriting causes data loss — lint guard enforces this. (2026-06-27: an agent used `>` instead of `>>`, lost 18+ lines. Now append-only is architectural.)

### Demand Management

On retrieval miss, agents write a demand file to `_meta/search-demands/<agent_id>/`:

```yaml
demand_id: <uuid>
agent_id: mama1
topic: "topic"
context: "trigger scenario"
priority: medium
```

`merge_demands` cron merges duplicates. Resolved demands move to agent directory.

### IMA Write Protocol (5-step Read-then-Write)

Editor Agent must follow 5 steps before writing to IMA:
1. Search if topic exists
2. Verify folder path
3. Check filename conflicts
4. Decide: append (conflict) or add (no conflict)
5. Execute write

This prevents permanent naming pollution in IMA (which has no rename/delete).

---

## Key Design Decisions

See [ADR/](adr/) for full records:

| # | Decision | Why |
|---|----------|-----|
| [001](adr/001-four-layer-retrieval.md) | Four-layer vs single DB | 3 platforms, different toolchains — can't share one backend |
| [002](adr/002-single-editor.md) | Single Editor vs multi-Editor | Prevents knowledge drift from inconsistent compilation |
| [003](adr/003-append-only-log.md) | Markdown log vs SQLite | All agents can grep/read; append-only = immutable history |
| [004](adr/004-ima-as-l1.md) | Keep IMA in chain vs remove | 446 legacy entries can't be abandoned; gradual migration to Vault |

---

## Evolution

| Version | Status | Change |
|---------|:------:|--------|
| v1 | Deprecated | Each agent with independent memory, no sharing |
| v2 | **Current** | Four-layer retrieval + demand management + activity-log |
| v3 | Planned | IMA → archive-only; Vault as sole source; trust decay auto-downgrade |

---

## Adopt It

### Minimal (1 day)
1. Create shared vault directory
2. Add 5-line reflex discipline to each agent's SOUL
3. Create `_meta/activity-log.md` (append-only)
4. Create `_meta/search-demands/` directory
5. Designate one Editor Agent

### Full (1 week)
Add: taxonomy.yaml, merge_demands cron, IMA sync, kb-notify, compile pipeline, cross-platform skill implementations.

### Hard Constraints
- **Append-only log** — overwrite = data loss, lint guard required
- **Single Editor** — prevents knowledge drift
- **Query before answer** — always walk the four layers
- **Miss → demand** — no knowledge blind spots

---

## License

MIT