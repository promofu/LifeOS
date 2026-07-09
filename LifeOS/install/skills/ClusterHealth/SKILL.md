---
name: ClusterHealth
description: Run a reproducible Kubernetes cluster stability check across all namespaces, classify findings into Critical/Warning/Info, and deliver a clear operational verdict — stable / watch / incident — with concrete next steps. USE WHEN health check, cluster health, stability check, post-rollout verification, "is the cluster stable", pods crashing, restart symptoms, CrashLoopBackOff triage, after a deploy. NOT FOR deploying or mutating a cluster (this is read-only assessment), single-pod debugging with no health-overview goal, or non-Kubernetes infra checks.
---

# ClusterHealth

Reproducible operational health assessment for a Kubernetes cluster. Read-only by default. Answers one question fast: **is this cluster stable, should I watch it, or is it an incident?**

## Resolve the check command (per project)

The health-check command differs per project — resolve it in this order:

1. **Project task** matching `k8s:<env>:cluster:health` (or the project's equivalent `mise`/make/script). Prefer it — it encodes the team's thresholds and namespaces.
2. **Fallback**: the built-in kubectl sweep below.

Run once **per target environment** (e.g. `dev`, `prod`). State which environments you checked.

## Routine

1. **Sweep** pod state across all namespaces, per environment:
   - Project task if present, else `kubectl get pods -A` (`-o wide` for node placement).
2. **Classify** each finding:
   - **Critical** — not `Running`, `CrashLoopBackOff`, `ImagePull*` errors, stuck non-ready.
   - **Warning** — currently `Ready` but elevated restarts (default threshold `>= 3`).
   - **Info** — `Completed` job pods (not an incident).
3. **Detail-check Warnings** — don't guess:
   - `kubectl -n <ns> get pod <pod> -o yaml` (events, conditions)
   - `kubectl -n <ns> logs <pod> --previous` (last crash cause)
4. **Verdict** — exactly one, always with concrete next steps:
   - **stable** — `critical = 0`; only `Completed` pods or purely historical restarts.
   - **watch** — `critical = 0` but Warning pods with *current* restarts → suggest monitoring / alert rule.
   - **incident** — `critical > 0` or restart count climbing in a short window → suggest fix ticket / escalation.

## Guardrails

- **Read-only by default.** No destructive cluster actions without explicit approval.
- For production, state plainly what is read-only vs. write *before* any write action.
- Don't commit or push as part of a health check unless asked.

## Gotchas

- **`Completed` ≠ problem.** Job/CronJob pods rest in `Completed`; counting them as failures fabricates incidents. Only workloads expected to be `Running` count toward Critical.
- **Historical vs. current restarts.** A high restart count on a now-`Ready` pod that hasn't restarted recently is history, not an incident. Check the restart *timestamp/trend*, not just the number — `watch` requires restarts still climbing.
- **`--previous` holds the crash truth.** Current logs show the fresh container; the crash cause lives in `logs --previous`. Skip it and you debug the wrong container.
- **Always `-A`.** `kubectl get pods` without `-A` hides the failing workload in another namespace — sweep all namespaces every time.
- **Prefer the project task over raw kubectl.** A project's own `k8s:<env>:cluster:health` encodes its thresholds and namespace set; raw kubectl can disagree with what the team considers healthy.
