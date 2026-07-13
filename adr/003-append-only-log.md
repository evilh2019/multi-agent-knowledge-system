# ADR-003: Append-Only Markdown Activity Log

**Status:** Accepted
**Date:** 2026-06-27
**Deciders:** mama1

## Context

Need to track all agent operations for audit, debugging, and cross-agent notification. The log must be readable by all agents across 3 platforms, and must be tamper-evident.

## Decision

Use an append-only Markdown file (`_meta/agent-activity-log.md`) with a 6-column pipe-delimited format, enforced by a lint guard.

## Alternatives Considered

### A. SQLite Database
- **Pros:** Rich query capabilities, structured data
- **Cons:** Requires SQL client on all platforms; OpenClaw agents can't easily query it; adds dependency
- **Rejected:** Cross-platform readability is the primary constraint

### B. Structured Logging (JSON Lines)
- **Pros:** Machine-parseable, structured
- **Cons:** Harder for humans to read; grep still works but less readable
- **Rejected:** Markdown table is both machine-parseable and human-readable

### C. Git-based Version History
- **Pros:** Built-in immutability
- **Cons:** Requires git on all platforms; commit overhead for every operation
- **Rejected:** Overhead too high for per-operation logging

## Consequences

- **Positive:** All agents can read with `grep`/`read_file`. Append-only = immutable history. Lint guard catches violations.
- **Negative:** Query capabilities are limited (grep only). Statistical analysis needs additional scripts.
- **Mitigation:** Current scale (~20 lines/day) is manageable. Dedicated analysis scripts can be added when needed.

## Incident

2026-06-27: An agent used `>` (overwrite) instead of `>>` (append), losing 18+ lines of history. This incident elevated append-only from a guideline to an architectural hard constraint with automated lint enforcement.