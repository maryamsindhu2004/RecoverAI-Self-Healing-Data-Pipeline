# RecoverAI — Self-Healing Data Pipelines

**Project title:** Learning-Based Data Pipeline Failure Diagnosis and Safe Automated Recovery
**Type:** Final Year Project (software only)
**Difficulty:** Moderate | **Competition potential:** Medium–High

---

## 1. Project Overview & Problem Statement

Modern data pipelines (ETL/ELT) feed dashboards, reports, and ML models. They fail often and in many ways:

- **Schema drift:** an upstream source adds, removes, renames, or retypes a column.
- **Missing values:** sudden null spikes in critical fields.
- **Duplicate records:** repeated loads or replayed batches.
- **Distribution shifts:** values that are valid in type but statistically abnormal.
- **Interrupted or failed transformations:** crashed tasks, timeouts, partial writes.

In most organizations a failed pipeline simply stops, and an engineer must read logs, find the cause, patch the pipeline, and re-run it manually. This is slow, error-prone, and delays every downstream consumer. Worse, quiet failures (e.g., a null spike that does not crash anything) can silently corrupt downstream data.

Self-healing pipelines are an active research area, and several AI-driven architectures have been proposed. Simply attaching an anomaly detector or an LLM to a pipeline is therefore not a meaningful contribution. The real open problem is **safety**: automatic repairs can themselves corrupt data, and plausible-looking LLM suggestions often fail in execution.

**This project addresses the question:** *Can a pipeline automatically diagnose a failure and repair it while guaranteeing that the repair does not violate data-quality rules or damage downstream consumers?*

---

## 2. Proposed Solution

RecoverAI is a local, software-only system that wraps a multi-stage ETL pipeline with a **detect → diagnose → propose → validate → commit-or-rollback** loop.

1. **Detect.** Monitor both the data (null rates, duplicates, schema, distributions) and pipeline execution metrics (runtime, row counts, task status) using rule-based checks and Isolation Forest.
2. **Diagnose.** Combine execution logs with a pipeline **dependency graph** to identify the likely root cause and the affected downstream tasks. Rules form the core; an LLM is an optional, clearly separated extension.
3. **Propose.** Select a repair from a **predefined catalogue of reversible actions** (e.g., add a missing column with a default, cast a type, deduplicate on a key, quarantine bad rows, retry with backoff, revert to the last good schema). Free-form generated code is never executed.
4. **Validate in a sandbox.** Apply the proposed repair to a copy of the data and re-run the affected downstream stages. Check the result against explicit data-quality rules (Great Expectations or custom assertions).
5. **Commit or roll back.** Commit only if all checks pass; otherwise roll back and escalate to a human with the full diagnosis.
6. **Audit.** Record the original data snapshot, diagnosis, attempted repair, validation results, and outcome in an immutable audit log.

**Example:** an upstream CSV gains a new column. RecoverAI detects schema drift, identifies the schema change as the cause, proposes a compatible schema update, tests the downstream transformations in the sandbox, and approves the change only if every downstream check passes.

**Design principle:** data-quality rules are declared in version-controlled YAML files and validated *before* any pipeline is generated or run, so invalid configurations fail early and never reach execution.

### Core novelty

The contribution is **safe recovery with data-quality guarantees**: restricting actions to reversible operations, requiring sandbox validation before commit, and measuring the *false-repair rate* as a first-class metric. The project compares this workflow against simpler recovery strategies instead of presenting it as better by assumption.

---

## 3. Objectives & Expected Outcomes

### Objectives

1. Build a multi-stage ETL pipeline with explicit, declarative data-quality rules.
2. Build a fault-injection framework that produces controlled, reproducible failures.
3. Implement rule-based and ML-based (Isolation Forest) detection baselines.
4. Implement dependency-graph-based root-cause diagnosis.
5. Implement a catalogue of reversible repair actions with sandbox validation and rollback.
6. Implement an audit trail and a monitoring dashboard.
7. Evaluate three recovery strategies: manual-style, rule-based, and the proposed diagnosis-and-validation workflow.

### Expected outcomes

- A working prototype demonstrating automatic detection, diagnosis, and safe recovery for the supported fault types.
- A quantitative comparison showing the trade-off between recovery rate, false-repair rate, and overhead.
- An open, reproducible fault-injection benchmark (synthetic and public datasets).
- A final report and demo suitable for FYP evaluation and student competitions.

### Evaluation metrics

| Metric | Purpose |
|---|---|
| Failure-detection precision, recall, F1 | Detection quality |
| Root-cause identification accuracy | Diagnosis quality |
| Mean time to detection (MTTD) / mean time to recovery (MTTR) | Speed |
| Successful recovery rate per fault type | Effectiveness |
| **False-repair rate** | Safety (incorrect automatic repairs) |
| Data-quality rule pass rate after recovery | Output validity |
| Pipeline runtime and computational overhead | Cost of the safety layer |

---

## 4. Target Users

- **Data engineers** who spend significant time on pipeline firefighting.
- **Analytics and BI teams** who depend on trustworthy, timely data.
- **ML engineers** whose models consume pipeline outputs.
- **Small teams and startups** without dedicated data-reliability staff.
- **Students and researchers** studying data reliability, AIOps, and data quality.

---

## 5. Key Features / Deliverables

1. **Sample ETL pipeline** (multi-stage) with YAML-defined data-quality rules and an inline config validator.
2. **Fault-injection toolkit** covering null spikes, schema drift, duplicate rows, distribution shift, and failed transformations.
3. **Detection module** with rule-based and Isolation Forest detectors, plus execution-metric monitoring.
4. **Diagnosis engine** using the pipeline dependency graph and execution logs (optional LLM-assisted explanations, never directly executed).
5. **Repair catalogue** of predefined, reversible actions.
6. **Sandbox validator** that tests repairs and downstream stages before commit.
7. **Rollback mechanism** with data snapshots.
8. **Audit trail** in PostgreSQL: original data reference, attempted repair, validation result, outcome.
9. **Dashboard** (FastAPI backend + React frontend) showing pipeline health, incidents, proposed repairs, approvals, and audit history.
10. **Evaluation report** comparing the three recovery strategies across all metrics.
11. **Dockerized setup** so the whole system runs locally with one command.

---

## 6. Implementation Plan & Timeline

The plan assumes a ~30-week FYP split over two semesters; adjust to the department's calendar.

| Phase | Weeks | Work | Output |
|---|---|---|---|
| 1. Research & scoping | 1–3 | Literature review of self-healing pipelines, data-quality tools, and anomaly detection; finalize fault types and datasets | Scope document, related-work summary |
| 2. Baseline pipeline | 4–7 | Build multi-stage ETL (Airflow/Prefect, pandas, SQL); write YAML data-quality rules and config validator | Running pipeline with passing quality checks |
| 3. Fault injection | 8–10 | Implement controlled fault injectors and a labelled failure dataset | Reproducible fault benchmark |
| 4. Detection | 11–14 | Rule-based and Isolation Forest detectors; execution-metric monitoring; tune and evaluate | Detection baselines with precision/recall/F1 |
| 5. Diagnosis | 15–18 | Dependency-graph builder, log parsing, root-cause rules; optional LLM explanation module | Diagnosis module with accuracy results |
| 6. Safe recovery | 19–23 | Repair catalogue, sandbox validation, rollback, audit trail | End-to-end recovery loop |
| 7. Dashboard & integration | 24–26 | FastAPI endpoints, React dashboard, Docker packaging | Demo-ready system |
| 8. Evaluation | 27–29 | Run all experiments across three strategies; analyze results and overhead | Results tables and plots |
| 9. Documentation & demo | 30 | Final report, poster, demo video, presentation | Final submission |

**Milestones:** M1 baseline pipeline (week 7), M2 detection working (week 14), M3 end-to-end recovery (week 23), M4 final evaluation (week 29).

---

## 7. Resources or Requirements

**Hardware:** a standard laptop/PC (8–16 GB RAM). No GPU required.

**Software and technology stack**

| Module | Technology |
|---|---|
| Pipeline orchestration | Apache Airflow or Prefect |
| Data processing | Python, pandas, SQL |
| Anomaly detection | scikit-learn (Isolation Forest) |
| Root-cause diagnosis | Rules, dependency graph (e.g., NetworkX), optional LLM |
| Validation and recovery | Great Expectations or custom assertions |
| Backend and dashboard | FastAPI, React |
| Storage and audit logs | PostgreSQL, Docker |

**Data:** synthetic data generated for the project, plus public datasets (e.g., open transactional or NYC taxi-style datasets) for realism.

**Other:** Git for version control; optional access to an LLM API for the diagnosis-explanation extension; supervisor guidance on data-engineering practices.

---

## 8. Potential Challenges or Risks

| Risk | Mitigation |
|---|---|
| **Automatic repairs corrupt data** | Restrict actions to predefined, reversible operations; mandatory sandbox validation; snapshots and rollback; escalate to a human when unsure |
| **Small set of simulated faults makes results unrealistic** | Use multiple fault types and severities, randomized injection, both synthetic and public datasets; state limitations honestly |
| **LLM recovery plans look plausible but fail** | LLM is optional and only explains or ranks; it never produces executable code; every action must come from the catalogue and pass validation |
| **Scope grows into a full cloud orchestration platform** | Keep the system local; fix the number of fault types and repair actions early; defer extras to future work |
| **Isolation Forest has false positives on normal drift** | Compare against rule-based baselines; tune thresholds; report precision and recall honestly |
| **Overhead of sandbox validation** | Measure it as an explicit metric; validate only affected downstream stages |
| **Difficulty showing novelty over existing work** | Focus the contribution on measurable safety (false-repair rate, post-recovery quality pass rate) and a controlled three-way comparison |

---

## 9. Additional Details (Optional)

**Scope boundaries (out of scope):** distributed or cloud-scale deployment, streaming systems, automatic repair of arbitrary code bugs, and repairs outside the predefined catalogue.

**Ethics and data handling:** only synthetic or public datasets are used; no private or sensitive data.

**Possible future work:** learning repair-action ranking from past incidents, extending to streaming pipelines, human-in-the-loop approval workflows, and integration with additional orchestrators.

**Why this project is a good FYP:** it is demonstrable end to end, has clear quantitative evaluation, combines data engineering, ML, and software design, and can be scoped up or down depending on time.
