---
name: Consolidate
description: "Recurring KVP consolidation pass over a knowledge cluster — gathers the related docs, surfaces overlaps, contradictions, and staleness, folds confirmed facts into the cluster's umbrella ISA (the single source of truth), and emits a prune-report of suggested merges and deletions for human approval. NEVER auto-deletes. Composes existing PAI primitives only (KnowledgeGraph traversal, Knowledge contradictions/develop, ISA Reconcile, SessionHarvester) — it does not build a parallel engine. Success metric is FEWER, SHARPER docs after the pass, not more. Single workflow: Consolidate. USE WHEN consolidate, konsolidieren, KVP pass, feedback loop over docs, dedupe notes, find overlapping notes, prune knowledge, fold scattered docs into one source of truth, summarize a topic cluster, re-converge a knowledge area, periodic knowledge review. NOT FOR creating a single new note (use Knowledge add), scaffolding or auditing an ISA in isolation (use ISA Scaffold/CheckCompleteness), general web research (use Research), or auditing instruction-set over-prompting (use BitterPillEngineering)."
effort: medium
---

# Consolidate — the KVP feedback loop over a knowledge cluster

PAI is good at *capturing* (SessionHarvester, KnowledgeHarvester) and *retrieving* (MemoryRetriever, KnowledgeGraph). What it lacks is an orchestrated, recurring **consolidation** pass — one that re-reads a cluster of related docs, finds what overlaps / contradicts / went stale, and converges them into a single sharp source of truth.

This skill IS that pass. It is the operational form of Mission **M0** ("Alles Gesammelte konsolidieren") and the **KVP** principle — applied to the knowledge base itself.

**It composes; it does not rebuild.** Every step delegates to an existing PAI primitive. The skill is the thin orchestration that chains them into a ritual. If a step here would duplicate a Knowledge or ISA capability, call that capability instead.

**Success is subtractive.** A successful run leaves the cluster with *fewer, sharper* documents — duplicates merged, stale notes flagged, contradictions resolved, the umbrella ISA tightened. A run that produces net-new sprawl has failed at its one job.

---

## The pass (one workflow)

| Step | Does | Delegates to |
|------|------|--------------|
| 1 Resolve | Find the cluster's docs (2-hop graph / tag-overlap / explicit list) | `KnowledgeGraph`, Knowledge `graph` |
| 2 Read | Load the docs (or their summaries) | Read / MemoryRetriever |
| 3 Diagnose | Overlaps · contradictions · staleness (frontmatter `updated`) · seedlings | Knowledge `contradictions`, Knowledge `develop` |
| 4 Converge | Fold confirmed facts into the umbrella ISA (SoT); link sources, don't copy them | ISA `Reconcile` / Edit |
| 5 Report | Emit a **prune-report**: suggested merges, deletions, stale-flags, contradictions-to-resolve | (proposal only) |
| 6 Apply | ONLY after human approval: apply approved prunes; log in ISA `## Decisions` / `## Changelog` | Edit |

Full procedure: `Workflows/Consolidate.md`.

---

## Guardrails

- **Proposes, never disposes.** The pass NEVER deletes or overwrites a note on its own. Step 5 produces a proposal; step 6 runs only on explicit approval (mirrors the Kanban weekly-review ritual).
- **The umbrella ISA is the SoT; source docs stay the detail.** Consolidation folds *claims* up into the ISA and *links* the sources — it does not copy their bodies into the ISA (that would just move the sprawl).
- **The prune-report is transient, not a 41st document.** Write it to the cluster's WORK dir as a working artifact; it is consumed and discarded, not archived as permanent knowledge.

---

## Gotchas

- **Dedupe against everything seen, not just what survived.** When iterating, track every doc already evaluated — not only the ones kept — or rejected/merged notes resurface every pass and the loop never converges.
- **Net-count is the gate.** Record doc-count before and after. If the pass didn't reduce or sharpen, say so honestly rather than declaring success — the whole point is anti-sprawl.
- **Compose, don't reimplement.** If you find yourself writing contradiction-detection or graph-traversal logic inline, stop — call Knowledge `contradictions` / `KnowledgeGraph`. This skill is orchestration, not engine.
- **Staleness is a flag, not a verdict.** An old `updated` date marks a doc *for review*, not for deletion — some facts are simply stable. Surface it; let the human decide.
- **Voice is mute-gated.** Any notification routes through the universal audio sink (`scripts/play-audio.sh`), which honors the global mute toggle. Do not synthesize speech directly.
```
