# MatchDay

An end-to-end MLOps project that predicts football match outcomes, explains its predictions, and evaluates them against actual results.

For each upcoming fixture, MatchDay will publish probabilities for a home win, draw, and away win. Predictions will be saved before kickoff so performance can be evaluated honestly afterward.

## Project status

**Planning only.** Application code, infrastructure, and trained models have not been implemented yet.

See [PLAN.md](PLAN.md) for the implementation checklist, phase status, completion criteria, and handoff notes. That file is the source of truth for progress.

## Intended experience

1. Open the dashboard and select an upcoming match.
2. View outcome probabilities and the most likely result.
3. Read explanations based on the model's actual inputs and feature contributions.
4. After the match, compare the saved prediction with the result.
5. Inspect aggregate model performance and prediction history.

Example output below is illustrative, not a real forecast:

| Outcome | Probability |
| --- | ---: |
| Home win | 58% |
| Draw | 25% |
| Away win | 17% |

An explanation might identify home performance, relative team strength, and recovery time as contributing signals. Explanations describe model behavior; they do not establish causes or guarantee an outcome.

## Initial scope

- One league, selected after checking data availability, history, licensing, and cost.
- Pre-match predictions, initially targeted at 24 hours before scheduled kickoff.
- Result after regulation time, including stoppage time; exclude extra time and penalties.
- Three probabilities that sum to one, plus the highest-probability outcome.
- Scheduled collection, prediction, evaluation, and candidate retraining.
- A prediction API, user dashboard, and operational monitoring.

The prediction horizon is a proposed default to confirm in phase 1. Predictions made late because of collection outages must be identified separately. Postponements and kickoff changes require explicit prediction versioning.

### Outside the first release

- In-play predictions, betting advice, and automated wagering.
- Exact-score, player-performance, and injury-prediction models.
- Claims about injuries, lineups, tactics, or expected goals without a verified source.
- Multiple leagues before the initial league is validated.
- LLM-generated explanations without grounded feature evidence.
- Kubernetes, a feature store, and other infrastructure without a demonstrated need.

## Proposed architecture

```text
Football data source
        |
        v
Airflow collection DAG --> Raw snapshots + PostgreSQL
                                    |
                              Feature pipeline
                                    |
                      Training DAG --> MLflow
                                    |
                         Evaluated model version
                                    |
                      Shared prediction service
                         /                   \
             Scheduled prediction DAG       FastAPI
                         \                   /
                          Saved predictions
                                 |
                              Dashboard
                                 |
Actual match results --> Evaluation DAG --> Performance history

Prometheus + Grafana: operational and published model metrics
Docker Compose: local service environment
CI/CD: checks, image builds, and controlled deployment
```

The scheduled predictor and API should share inference code. Durable prediction records, rather than service memory, anchor explanations and evaluation.

## Technology responsibilities

| Component | Responsibility |
| --- | --- |
| Python | Collection, validation, feature engineering, training, and evaluation |
| Airflow | DAG dependencies, schedules, retries, backfills, and run visibility |
| PostgreSQL | Normalized fixtures, results, feature metadata, and prediction history |
| Raw artifact storage | Immutable source responses, dataset snapshots, and evaluation artifacts; local storage initially, object storage if needed |
| MLflow | Experiment tracking, artifacts, model registration, and model lineage |
| scikit-learn and a selected tree library | Baselines, candidate models, and probability calibration where justified |
| SHAP or a validated equivalent | Feature contributions translated into grounded explanations |
| FastAPI | Prediction and history endpoints, validation, and health checks |
| Dashboard framework, to be selected | Fixtures, explanations, prediction history, and performance views |
| Prometheus | Service, ingestion, and pipeline metrics |
| Grafana | Operational dashboards and alerts |
| Docker / Compose | Reproducible local services and deployable application images |
| CI/CD provider, to be selected | Automated checks, builds, and deployment controls |

HTML scraping is optional, not a requirement. Prefer an official or permitted API when it provides the data. Verify collection terms, attribution, rate limits, and historical coverage before implementation.

## Modeling and evaluation

Candidate inputs include team-strength ratings, rolling form, home/away performance, scoring history, and rest days. Availability depends on the selected source.

- Start with a simple historical-frequency baseline and a team-strength baseline.
- Compare logistic regression with a suitable tree-based classifier.
- Evaluate chronologically, with a separate final test period.
- Calculate rolling features using only information available at prediction time.
- Track log loss, multiclass Brier score, calibration, and accuracy.
- Document the Brier-score convention and evaluate baseline and candidate on identical fixtures.
- Assess the model's highest-probability outcome separately from the quality of its probability estimates.
- Fit preprocessing and any calibration using training/validation data only.

Historical backtests must distinguish reconstructed match history from genuinely archived, as-of source data. Do not claim a strict historical simulation where source availability cannot be established.

Retraining produces a candidate. Promotion requires documented evaluation criteria and an available rollback model. Keeping the baseline is acceptable if candidates fail to improve it.

## Reliability and observability

- Idempotent collection with stable fixture IDs and bounded retries.
- Data validation, quarantined invalid records, and freshness checks.
- Versioned datasets, models, predictions, and explanation artifacts.
- Reconciliation for postponed, cancelled, abandoned, and corrected matches.
- Alerts for stale feeds, pipeline failures, and service errors.
- Model metrics calculated after results arrive, with sample counts and evaluation windows.
- Low-cardinality Prometheus labels; detailed fixture history belongs in the database.
- Secrets supplied through environment/configuration mechanisms and never committed.

## Development and execution

Setup commands will be added after the repository structure and dependencies exist. There is currently no runnable application.

Implementation begins with phase 1 in [PLAN.md](PLAN.md). Introduce Docker and automated checks incrementally; the final deployment phase verifies the integrated system.

## Definition of project completion

The project is complete when a deployed installation has collected fresh data, published a prediction before a match, recorded the actual result, evaluated the saved prediction, and exposed operational metrics. Reproducible setup instructions, evaluation evidence, monitoring, and rollback/recovery procedures must also be documented.
