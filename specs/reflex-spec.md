# Reflex Spec — Multi-Agent Knowledge Retrieval

> The 5-line discipline every agent must follow.

## SOUL Discipline (5 lines, hard constraint)

Each agent's SOUL.md must include:

```markdown
## Knowledge Reflex (multi-agent KB v2)

- Query: L3 memory → L2 vault → L1 IMA → L0 external
- Answer: always tag source (hit layer + file path)
- Miss: never guess — write demand to _meta/search-demands/<self>/
- Correction: log to source (trust=0.95 + corrections appended)
- Cross-domain: UX+AI → OscarMa1; everything else → mama1
```

## Retrieval Interface

Every agent's reflex skill must implement:

```
knowledge_reflex(query, agent_id) → {
    hit_layer: "L3" | "L2" | "L2.5" | "L1" | "L0" | None,
    content: str | None,
    source_path: str | None,
    demand_id: str | None
}
```

## Platform Implementations

| Platform | Agent | Implementation |
|----------|-------|---------------|
| Hermes | mama1, OscarMa1 | `hermes skill run knowledge-reflex` |
| OpenClaw | XieXie1, OscarXia1 | `skills/knowledge-reflex/SKILL.md` |
| StepCode | CC1 | Read+grep+Write (no IMA access) |

**CC1 special:** Skips L1 (IMA) and L0 (external). L3→L2 only. Miss → write demand.

## Activity Log Format

```
| YYYY-MM-DD | agent_id | reflex | hit_layer | OK | query_summary → source_path |
```

**Append-only.** Lint guard enforces. Never use `>` or `open("w")`.