# MatchDay implementation plan

## How agents must maintain this plan

This document is the source of truth for implementation progress.

1. Read this plan and the repository instructions before starting work.
2. Set the active phase to `IN PROGRESS` in the status table.
3. Mark an individual task `[x]` only after its implementation and relevant verification are complete.
4. Mark a phase `DONE` and check its phase checkbox only after every task and its completion gate are satisfied.
5. Fill in that phase's completion record with the date, files/artifacts, verification results, and material limitations. Do not claim checks that were not run.
6. If work is interrupted, leave incomplete tasks unchecked and update the handoff section with the exact next action.
7. If a dependency prevents progress, use `BLOCKED`, explain the dependency, and identify work that can continue. Do not mark blocked work complete.
8. Update the README when setup, architecture, scope, or user-facing behavior changes. Keep phase status here rather than maintaining duplicate progress tables.
9. Record scope changes and their reasons under Decisions. If a requirement is removed, explicitly amend the task and gate; do not silently check it off.

Allowed phase statuses: `NOT STARTED`, `IN PROGRESS`, `BLOCKED`, `DONE`.

## Phase status

| Phase | Name | Status | Completed on |
| --- | --- | --- | --- |
| 1 | Scope and data feasibility | IN PROGRESS | — |
| 2 | Data collection and storage | NOT STARTED | — |
| 3 | Features and training datasets | NOT STARTED | — |
| 4 | Modeling and MLflow | NOT STARTED | — |
| 5 | Prediction API and explanations | NOT STARTED | — |
| 6 | Airflow automation | NOT STARTED | — |
| 7 | Dashboard and monitoring | NOT STARTED | — |
| 8 | Deployment and release hardening | NOT STARTED | — |

## Phase 1 — Scope and data feasibility

- [ ] **Phase complete**
- [ ] Select one league and inspect representative historical and upcoming fixtures.
- [ ] Verify source access, terms, attribution, rate limits, history depth, fields, and cost; document findings.
- [ ] Confirm that enough completed matches and trustworthy timestamps exist for chronological evaluation; record actual counts and date coverage.
- [ ] Define the outcome labels and handling of extra time, penalties, postponements, cancellations, abandoned matches, and corrections.
- [ ] Confirm the prediction horizon, schedule/timezone, and treatment of late predictions and changed kickoff times.
- [ ] Select initial storage, dashboard, model-library, and deployment choices within a documented resource budget.
- [ ] Create the repository structure, dependency configuration, configuration example without secrets, and architecture/data-contract notes.

**Completion gate:** A small source-access spike demonstrates usable data. Scope, data constraints, target definition, and initial architecture are documented. No unsupported source or feature assumptions remain in the initial scope.

**Completion record:**
- Completed on: —
- Files/artifacts: —
- Verification and results: —
- Limitations/deviations: —

## Phase 2 — Data collection and storage

- [ ] **Phase complete**
- [ ] Implement the source client with pagination, rate-limit handling, bounded retries, timeouts, and incremental collection.
- [ ] Preserve raw responses with retrieval timestamps and source metadata.
- [ ] Create database migrations and normalized team, competition, fixture, and result records.
- [ ] Retain fixture schedule/status changes instead of silently losing prediction context.
- [ ] Validate required fields, identifiers, timestamps, scores, and statuses; quarantine invalid records.
- [ ] Implement idempotent upserts and collection checkpoints.
- [ ] Backfill the agreed history and collect upcoming fixtures.
- [ ] Verify reruns, interrupted-run recovery, malformed responses, and a fresh incremental update.
- [ ] Add reproducible database/container setup and collection instructions.

**Completion gate:** Historical and new data reach storage reproducibly. Repeating a collection does not duplicate records, and failures do not silently discard or corrupt data.

**Completion record:**
- Completed on: —
- Files/artifacts: —
- Verification and results: —
- Limitations/deviations: —

## Phase 3 — Features and training datasets

- [ ] **Phase complete**
- [ ] Implement shared, versioned feature definitions for training and inference.
- [ ] Calculate team-strength ratings, rolling form, home/away history, scoring statistics, and rest days where supported.
- [ ] Define fallbacks for promoted/new teams and insufficient history.
- [ ] Enforce prediction cutoffs using event times and known source-availability times; document historical reconstruction limits.
- [ ] Verify that later results cannot change earlier feature values.
- [ ] Build chronological training, validation, and reserved final-test partitions, with documented date boundaries.
- [ ] Save dataset manifests with source snapshot references, feature versions, counts, cutoffs, and reproducibility identifiers.
- [ ] Verify online/offline feature consistency on representative fixtures.

**Completion gate:** A dataset can be rebuilt from recorded inputs. Temporal checks demonstrate that features exclude future results and use the same definitions as inference.

**Completion record:**
- Completed on: —
- Files/artifacts: —
- Verification and results: —
- Limitations/deviations: —

## Phase 4 — Modeling and MLflow

- [ ] **Phase complete**
- [ ] Train historical-frequency and team-strength baselines.
- [ ] Train logistic regression and a selected tree-based candidate.
- [ ] Track parameters, seeds, dataset/feature versions, code revision, metrics, and artifacts in MLflow.
- [ ] Evaluate chronological validation windows using log loss, a documented multiclass Brier convention, calibration, accuracy, and sample counts.
- [ ] Fit preprocessing and any probability calibration without using test data.
- [ ] Define candidate promotion criteria before final comparison, including data-quality and probability-validity checks.
- [ ] Select the initial model honestly; retain a baseline if candidates are inferior.
- [ ] Run the reserved final-test evaluation after selection and publish a model report with limitations.
- [ ] Register the selected model and verify reload/inference with fixed class ordering and probabilities summing to one.

**Completion gate:** Tracked experiments and a reproducible evaluation report support the chosen model. Its artifact reloads successfully and its lineage is recorded.

**Completion record:**
- Completed on: —
- Files/artifacts: —
- Verification and results: —
- Limitations/deviations: —

## Phase 5 — Prediction API and explanations

- [ ] **Phase complete**
- [ ] Implement FastAPI endpoints for fixtures, prediction retrieval/creation, history, health, and readiness as appropriate.
- [ ] Load a pinned deployed model version and reuse the shared feature pipeline.
- [ ] Return home/draw/away probabilities, selected outcome, prediction timestamp, kickoff time, horizon, and model version.
- [ ] Generate validated feature-based explanations with readable templates and documented reference values.
- [ ] Keep explanation wording grounded in recorded inputs; handle missing explanations without inventing reasons.
- [ ] Persist immutable prediction versions, feature snapshots/references, and explanation artifacts; explicitly handle repeats and kickoff changes.
- [ ] Handle missing history, stale data, unavailable models, invalid requests, and already-started fixtures.
- [ ] Verify inference, persistence, probability ordering, explanation consistency, and failure responses; add API container and usage instructions.

**Completion gate:** An upcoming fixture produces a valid, stored, reproducible prediction and grounded explanation through the API. Historical predictions remain unchanged after new results arrive.

**Completion record:**
- Completed on: —
- Files/artifacts: —
- Verification and results: —
- Limitations/deviations: —

## Phase 6 — Airflow automation

- [ ] **Phase complete**
- [ ] Implement the collection DAG with incremental windows, validation, and source freshness checks.
- [ ] Implement the prediction DAG with horizon eligibility, shared inference, and duplicate/version controls.
- [ ] Implement the evaluation DAG joining settled results to eligible saved predictions; handle corrections and unresolved matches.
- [ ] Implement the training DAG with dataset creation, MLflow logging, candidate evaluation, and controlled promotion.
- [ ] Separate replay/backtest predictions from genuinely published pre-match predictions in storage and metrics.
- [ ] Configure dependencies, timezones, retries, timeouts, concurrency, catchup/backfill behavior, and failure reporting.
- [ ] Require enough new labeled data before retraining; do not promote on a schedule alone.
- [ ] Verify DAG loading and exercise reruns, bounded backfills, partial failure recovery, and one integrated cycle.
- [ ] Add Airflow services to the container setup and document operations.

**Completion gate:** DAGs collect, predict, reconcile results, and evaluate without manual task execution. Reruns are safe and candidate promotion follows the recorded policy.

**Completion record:**
- Completed on: —
- Files/artifacts: —
- Verification and results: —
- Limitations/deviations: —

## Phase 7 — Dashboard and monitoring

- [ ] **Phase complete**
- [ ] Display upcoming fixtures, probabilities, explanations, freshness, and model versions.
- [ ] Display prediction history, actual outcomes, evaluation windows, sample counts, and baseline comparisons.
- [ ] Provide clear empty, stale, pending-result, postponed, and error states.
- [ ] Instrument API traffic, latency, errors, readiness, ingestion freshness, invalid records, and pipeline failures.
- [ ] Publish delayed model-performance and drift summaries with documented windows; keep high-cardinality details out of Prometheus labels.
- [ ] Provision Prometheus and Grafana configuration and dashboards in version control.
- [ ] Configure actionable alerts for stale collection, failed pipelines, and service problems; handle insufficient model-evaluation samples.
- [ ] Verify visible dashboard behavior and trigger/recover a controlled failure to demonstrate monitoring.

**Completion gate:** The user interface explains real system output, while monitoring detects a demonstrated operational failure and its recovery. Reported performance comes from saved predictions and settled outcomes.

**Completion record:**
- Completed on: —
- Files/artifacts: —
- Verification and results: —
- Limitations/deviations: —

## Phase 8 — Deployment and release hardening

- [ ] **Phase complete**
- [ ] Finalize Docker images, Compose configuration, persistent volumes, health checks, dependency locking, and secret handling.
- [ ] Implement CI checks and image builds; configure deployment for the chosen environment.
- [ ] Configure appropriate access controls for administrative services, model management, and prediction-triggering endpoints.
- [ ] Document source configuration, credentials, installation, migrations, startup, shutdown, and resource requirements.
- [ ] Verify backup/restore for durable data and rollback to a previous model/application version.
- [ ] Verify restart recovery, upstream outage behavior, and serving under a documented representative load.
- [ ] Deploy and record a fresh prediction made before kickoff, its later result, and the resulting evaluation.
- [ ] Finish architecture documentation, model/data reports, operating runbook, screenshots, and a reproducible demo.
- [ ] Write resume/project claims using measured results only; document remaining limitations.

**Completion gate:** A deployed system completes the real pre-match prediction-to-evaluation cycle. Setup, monitoring, recovery, and rollback are demonstrated and documented. A replay alone does not satisfy the live-cycle requirement.

**Completion record:**
- Completed on: —
- Files/artifacts: —
- Verification and results: —
- Limitations/deviations: —

## Decisions

| Date | Decision | Reason / consequences |
| --- | --- | --- |
| 2026-10-30 | Use eight implementation phases | Separate data, modeling, serving, automation, monitoring, and release gates |
| 2026-10-30 | Start with one league and regulation-time outcomes | Keep the initial target and evaluation tractable |
| 2026-10-30 | Propose predictions 24 hours before kickoff | Establish a consistent pre-match information cutoff; confirm feasibility in phase 1 |
| 2026-10-30 | Leave provider and league selection open | Verify access and historical coverage before committing |

## Current handoff

- **Last completed work:** Initial README and implementation plan created. No implementation phase has been completed.
- **Active phase:** Phase 1 - source access and scope verification in progress.
- **Next action:** Inspect candidate data providers and sample one league's historical/upcoming fixtures; document coverage, cost, permissions, and available timestamps.
- **Known blockers:** None established; provider access and budget remain unverified.
- **Verification performed:** Documentation structure and consistency only; no application or model checks exist yet.

## Progress log

Append an entry after each meaningful implementation session. Keep detailed results in the relevant completion record or linked artifact.

| Date | Phase | Work completed | Verification | Next step |
| --- | --- | --- | --- | --- |
| 2026-10-30 | Planning | Created project README and trackable eight-phase plan | Documentation review | Begin data feasibility assessment |
