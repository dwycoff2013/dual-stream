
# dualstream_anticollapse

A small, **functional** reference toolkit that operationalizes the practices from
**Preventing Model Collapse in Production AI: A Practical Guide** and connects them to the
**Dual-Stream Architecture** (Answer Stream + Monologue Stream) with a **Coherence Audit**.

## What you get

- Drift detection (PSI + KS) and a simple **Page–Hinkley** concept-drift detector
- Performance monitoring (accuracy, precision/recall, F1, AUC/log-loss when available)
- Alert emission (stdout or file)
- Retraining triggers (scheduled or performance/drift-triggered)
- Model governance (hashing, registry)
- **Dual-Stream Coherence Auditor** that inspects the monologue for deception/safety/conflict markers and
  flags incoherence between what the model *thinks* vs what it *says*.

## Quickstart (in this folder)

```bash
# 1) Train a baseline on the demo reference data
python -m dualstream_anticollapse.cli train --train_csv demo/reference.csv --target y --features x1,x2 --artifacts artifacts

# 2) Monitor a new batch for drift + perf regression
python -m dualstream_anticollapse.cli monitor --reference_csv demo/reference.csv --current_csv demo/current.csv --target y --features x1,x2 --artifacts artifacts

# 3) Audit Dual-Stream outputs (JSONL)
python -m dualstream_anticollapse.cli audit-dual --dual_jsonl demo/dual_stream_sample.jsonl --artifacts artifacts
# => writes artifacts/coherence_report.json and emits coherence_violation events to stdout
```

## Data format for Dual-Stream audit

Each line in the JSONL must have:
```json
{"answer": "string", "monologue": "string with [TOKENS]", "logits_topk": [["token", 0.85], ...]}
```
The monologue is expected to contain probe-style blocks like:
- `[ATTN_HEAD_12.5_PROBE:TRACKING_SUBJECT]`
- `[MLP_LAYER_22_PROBE:ETHICAL_CONFLICT_DETECTED]`
- `[LOGIT_LENS:TOP_5:("sorry",0.85),("cannot",0.10), ... ]`

## Notes

- The thresholds and marker lists are small, readable rules so to be extended later.
- The Page–Hinkley detector provides an online signal for sudden performance shifts.
- All artifacts are placed under `artifacts/` by default.
