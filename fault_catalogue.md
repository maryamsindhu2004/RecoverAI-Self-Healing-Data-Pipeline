# Fault Catalogue

Controlled faults to inject into the sample ETL pipeline. Each fault is parameterized (severity, location, timing) so experiments are reproducible.

| ID | Fault | How it is injected | Expected root cause | Candidate repair (reversible) | Validation check |
|---|---|---|---|---|---|
| F1 | Null spike | Set a random 5–40% of a critical column to null | Upstream data quality drop | `quarantine_bad_rows` or fill from last good batch | Null rate within threshold; row count preserved |
| F2 | Schema drift: added column | Add an unexpected column to the source | Upstream schema change | `add_missing_column_with_default` / schema update | Downstream transforms run; schema matches contract |
| F3 | Schema drift: renamed or removed column | Rename or drop a required column | Upstream schema change | `revert_to_last_good_schema` or column mapping | All dependent stages succeed |
| F4 | Type change | Change a numeric column to string values | Upstream type change | `cast_column_type` | Casts succeed with no value loss |
| F5 | Duplicate rows | Replay a batch or duplicate 1–20% of rows | Duplicate load | `deduplicate_on_key` | Primary-key uniqueness holds; no valid rows lost |
| F6 | Distribution shift | Scale or offset values in a numeric column | Upstream data change | Flag and escalate (no auto-fix) | Human review required |
| F7 | Failed transformation | Raise an error mid-task | Transient or code failure | `retry_with_backoff` | Task completes; output matches expected |
| F8 | Partial or interrupted write | Kill a task mid-load | Interrupted execution | Rollback to snapshot, then retry | Row counts match source |

## Notes

- F6 is deliberately **not** auto-repaired. It tests whether the system correctly escalates instead of making a risky change.
- Combine faults (e.g., F1 + F5) in a subset of runs to test diagnosis under ambiguity.
- Include clean runs with no fault, to measure false positives and false repairs.
- Seed all random injections so results are reproducible.
