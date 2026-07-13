# ADR-001: Four-Layer Progressive Retrieval

**Status:** Accepted
**Date:** 2026-06-26
**Deciders:** mama1, cc1, Oscar

## Context

5 AI agents across 3 platforms (Hermes, OpenClaw, StepCode) need to share knowledge. Each platform has different tools, different memory systems, and different runtime constraints.

## Decision

Use a four-layer progressive retrieval model (L3→L2→L2.5→L1→L0) instead of a single shared database.

Each layer is independent and can be queried by any agent using platform-native tools. The retrieval order is fixed: L3 first (fastest), L0 last (slowest).

## Alternatives Considered

### A. Single Vector Database (Pinecone/Weaviate)
- **Pros:** Single source of truth, fast semantic search
- **Cons:** Requires network access from all agents; OpenClaw and StepCode agents can't easily call the same API; adds infrastructure dependency
- **Rejected:** Cross-platform compatibility is the primary constraint

### B. Git-based Knowledge Repo Only
- **Pros:** Simple, version-controlled
- **Cons:** No semantic search; agents must grep/read every file; slow for large vaults
- **Rejected:** Doesn't capture "what we talked about recently" (L3's value)

### C. L2→L3 Order (Vault First)
- **Pros:** Vault is more authoritative
- **Cons:** Most queries are about recent conversations, which L3 captures better; L2 is slower (file I/O)
- **Rejected:** L3-first gives better UX for the common case

## Consequences

- **Positive:** Each layer can evolve independently. No single point of failure. Agents only need platform-native tools.
- **Negative:** Retrieval latency is higher (up to 4 calls on full miss). Cross-layer consistency requires governance.
- **Mitigation:** demand management + merge_demands cron handles the consistency gap.