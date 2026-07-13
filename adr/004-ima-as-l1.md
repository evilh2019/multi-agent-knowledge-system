# ADR-004: Keep IMA as L1 in Retrieval Chain

**Status:** Accepted (with planned sunset)
**Date:** 2026-07-04
**Deciders:** mama1, Oscar

## Context

IMA (Feishu Knowledge Base) contains 446 legacy entries accumulated over years of use. It's Oscar's primary knowledge management tool. The new knowledge system (Vault + Hindsight) is more structured, but we can't abandon 446 entries overnight.

## Decision

Keep IMA as L1 in the retrieval chain, with a planned gradual migration to Vault. IMA remains the "raw archive" layer; Vault is the "compiled knowledge" layer; Hindsight is the "semantic index" layer.

## Alternatives Considered

### A. Drop IMA Immediately
- **Pros:** Simpler architecture, faster retrieval
- **Cons:** Loses 446 entries; Oscar's daily workflow disrupted
- **Rejected:** Too disruptive, too much data loss

### B. One-time Bulk Migration
- **Pros:** Clean break, single architecture
- **Cons:** 446 entries is too much for one batch; quality control impossible; IMA has no bulk export
- **Rejected:** Not feasible given IMA's API limitations

### C. IMA as Primary, Vault as Cache
- **Pros:** No workflow change for Oscar
- **Cons:** IMA has no rename/delete, no semantic search, no version control — poor foundation for knowledge system
- **Rejected:** Vault's structure is the long-term foundation

## Consequences

- **Positive:** Preserves 446 entries. Gradual migration reduces risk. Oscar's workflow unchanged.
- **Negative:** IMA retrieval is slow (network call). IMA has no rename/delete — naming pollution is permanent. Dual-source adds complexity.
- **Mitigation:** L3→L2→L1 order means IMA is only queried when Vault misses. IMA Write Protocol (5-step Read-then-Write) prevents further pollution. v3 roadmap: IMA → archive-only.

## Sunset Plan (v3)

1. Tag all IMA entries with taxonomy mapping
2. Batch-compile high-value entries to Vault
3. Set IMA to read-only (no new writes)
4. Remove IMA from reflex chain, keep as archive reference