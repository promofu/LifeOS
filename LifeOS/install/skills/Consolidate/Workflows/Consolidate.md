# Consolidate Workflow

Run one KVP consolidation pass over a knowledge cluster and emit a prune-report for approval.

## When to invoke

- User: `Skill("Consolidate", "<topic-or-umbrella-ISA-path>")`
- Scheduled (later, F5): a systemd-timer on the always-on host invokes this skill directly — no mise-task wrapper needed.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| cluster | yes | A topic, a tag, an explicit doc list, or an umbrella-ISA path. The anchor for what to consolidate. |
| umbrella_isa | no | Path to the cluster's source-of-truth ISA. If absent, identify or propose one (the cluster should have exactly one SoT). |
| apply | no | Default false. When true, apply the prune-report's *approved* items. The first emit is always a proposal. |

## Procedure

### Step 1 — Notification (mute-gated)

Output the text line; route any voice through `scripts/play-audio.sh` (honors the mute toggle — do not synthesize directly):

```
Running the **Consolidate** workflow to converge the **<cluster>** knowledge cluster...
```

### Step 2 — Resolve the cluster

Build the document set from the anchor:
- umbrella-ISA path → read its Problem/source links + run `KnowledgeGraph` 2-hop from there.
- topic/tag → Knowledge `graph` / tag-overlap to gather related notes.
- explicit list → use as-is.

Record the **starting doc count** and the file list. This is the baseline for the net-count gate.

### Step 3 — Read

Load each doc (or its MemoryRetriever summary for large ones). Note each doc's frontmatter `updated`.

### Step 4 — Diagnose

For the set, produce four findings lists:
- **Overlaps** — the same claim asserted in ≥2 docs (candidate merges).
- **Contradictions** — call Knowledge `contradictions`; conflicting claims across the set.
- **Stale** — `updated` older than the cluster's review threshold (flag for review, not deletion).
- **Seedlings** — call Knowledge `develop`; thin notes that should be enriched or absorbed.

### Step 5 — Converge into the umbrella ISA (SoT)

Fold confirmed, settled facts UP into the umbrella ISA and **link** the source docs (never copy their bodies). If the cluster has no ISA, propose scaffolding one via `Skill("ISA", "scaffold ...")` and stop for approval.
- Structural changes to the ISA (new/split ISCs, Goal refinement) → use the `refined:` prefix in `## Decisions` (ID-stability rule: never renumber).
- Mechanical check-offs from completed work → `Skill("ISA", "reconcile ...")` where an ephemeral exists.

### Step 6 — Emit the prune-report (PROPOSAL ONLY)

Write `prune-report.md` to the cluster's `MEMORY/WORK/<slug>/` dir — a **transient working artifact**, not permanent knowledge. Sections:
- **Merge** — which docs collapse into which (with the surviving target).
- **Delete (suggested)** — redundant/absorbed docs, each with the reason.
- **Stale — review** — docs whose `updated` crossed the threshold.
- **Contradictions — resolve** — conflicting claims needing a human call.
- **Net count** — starting N → projected M after approved prunes.

**Stop here on the first run.** The report is a proposal; the human approves.

### Step 7 — Apply (only with `apply: true` + explicit approval)

Apply ONLY the approved items:
- Merge: move the surviving content, then delete the absorbed source(s).
- Delete: remove approved-for-deletion docs.
- Log every applied change in the umbrella ISA `## Decisions`; add a `## Changelog` C/R/L entry if a belief changed.
- Update the cluster's MEMORY index (e.g. `MEMORY.md`) to drop removed pointers.
- Re-record the **final doc count** and confirm it dropped or held (never silently grew).

### Step 8 — Return

Report: starting count → final count, items merged/deleted/flagged, and the umbrella-ISA path. If net count did not improve, say so plainly.

## Guardrails (repeated — load-bearing)

- NEVER delete in step 6. Deletion happens only in step 7, only on approved items.
- The umbrella ISA holds *claims + links*, not copied bodies.
- Dedupe against all docs *seen* across iterations, not just those kept — else rejected notes resurface and the loop never converges.

## Failure modes

- **No SoT for the cluster** → propose scaffolding one ISA; do not consolidate into a vacuum.
- **Net count grew** → the pass failed its purpose; report honestly, don't dress it up.
- **Inline re-implementation** → if writing contradiction/graph logic by hand, stop and call the PAI primitive.
