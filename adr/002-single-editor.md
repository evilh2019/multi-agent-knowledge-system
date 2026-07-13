# ADR-002: Single Editor Agent

**Status:** Accepted
**Date:** 2026-06-28
**Deciders:** mama1, Oscar

## Context

With 5 agents sharing a vault, multiple agents could write compiled notes, source cards, and taxonomy entries. Without coordination, the same concept could be compiled differently by different agents, creating knowledge drift.

## Decision

Designate one Editor Agent (mama1) as the sole full-writer to the vault. Other agents write only in scoped domains (OscarMa1: UX+AI; XieXie1: personal proposals; OscarXia1: work proposals). CC1 is read-only.

## Alternatives Considered

### A. Multi-Editor with Merge
- **Pros:** Higher write throughput, no bottleneck
- **Cons:** Merge conflicts, inconsistent compilation quality, knowledge drift
- **Rejected:** Knowledge consistency > write throughput for this scale

### B. CRDT-based Conflict-Free Writes
- **Pros:** No bottleneck, theoretically no conflicts
- **Cons:** Overkill for 5 agents; CRDTs don't solve semantic conflicts (same concept, different framing)
- **Rejected:** Semantic consistency is the problem, not syntactic conflicts

## Consequences

- **Positive:** Single source of truth for compiled knowledge. Consistent quality. Clear accountability.
- **Negative:** Editor Agent is a bottleneck. If Editor is unavailable, knowledge accumulation stalls.
- **Mitigation:** Demand queue buffers unhandled requests. merge_demands cron batches processing. Domain Writers can still write in their scoped areas.