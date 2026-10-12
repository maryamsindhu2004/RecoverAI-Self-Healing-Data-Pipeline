# Evaluation Plan

## Research question

Does a diagnosis-and-validation recovery workflow improve safe recovery compared with simpler strategies, and at what cost?

## Strategies compared

| Strategy | Description |
|---|---|
| S1: Manual-style | Pipeline stops on failure; recovery simulated with a fixed delay plus a scripted human fix |
| S2: Rule-based | Fixed rule maps symptom directly to repair; no sandbox validation |
| S3: RecoverAI | Detect, diagnose via dependency graph, select from catalogue, sandbox validate, commit or roll back |

## Experimental setup

- **Datasets:** synthetic orders data plus at least one public dataset.
- **Faults:** F1–F8 from `fault_catalogue.md`, several severities each.
- **Runs:** repeated with different random seeds; include clean runs and mixed-fault runs.
- **Environment:** local Docker setup, identical for all strategies.

## Metrics

| Metric | Definition |
|---|---|
| Detection precision / recall / F1 | Correct anomaly flags vs. injected faults and clean runs |
| Root-cause accuracy | Fraction of incidents where the diagnosed cause matches the injected one |
| MTTD | Mean time from fault injection to detection |
| MTTR | Mean time from detection to validated recovery |
| Recovery success rate | Incidents fully recovered, per fault type |
| **False-repair rate** | Repairs committed that were incorrect or that left data invalid |
| Post-recovery rule pass rate | Share of data-quality rules passing after recovery |
| Overhead | Added runtime and CPU/memory from monitoring and sandbox validation |

## Analysis

- Report results per fault type and per strategy, with mean and spread across seeds.
- Compare detection baselines (rules vs. Isolation Forest) separately.
- Examine failure cases: which faults are misdiagnosed, and why.
- Report the safety/overhead trade-off honestly, including cases where S3 escalates instead of repairing.

## Threats to validity

- Injected faults may not match real-world failures; mitigate with varied severities, mixed faults, and public data.
- Small catalogue limits generality; state it as a scope boundary.
- Manual-style baseline is simulated; document the assumptions.
