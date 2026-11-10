---
name: vstack-s4-calibrate-tv
description: "Calibrate buggy/fixed/ablated behaviors and validate model conformance using real traces, witnesses and explicit mismatch triage."
---

# S4 — Calibrate and align traces

Read [Layer 0: shared operating rules](../../docs/layer-0.md), this stage and the assigned input bundle.

## Role and write scope

Model and trace workers own distinct outputs; the coordinator consolidates the checklist and independently reruns representatives.

## 1. Input

S3 (build an anchored model) models, configurations, source anchors and witnesses.

## 2. Procedure

1. Timestamp predictions. Run four-way calibration: historical bug violates the expected property by the expected mechanism; fix completes within bounds; each removed repair component violates; reachability witnesses are reached (their negations fail).
2. Repeat two-sided contract calibration and the minimum-scale/+one checks. A component outside the abstraction is unverified, not a pass.
3. Apply [trace-conformance checklist](../../docs/trace-validation.md) to real traces. Keep observation, instrumentation, completion and safety evidence separate.
4. Classify every NOT_CONSUMED at the first divergence; never edit a trace to make it count as accepted.
5. Review every run with [the seven questions](../../docs/run-review.md).

## 3. Exit checks

- Bug/fix/ablation and paired witnesses have expected verdicts and mechanisms. Generic mechanism and action-coverage automation is missing.
- Every trace checklist item has evidence and an honest covered/uncovered scope.
- Coordinator clean reruns agree. Timeout, unknown, empty exploration and zero required coverage cannot pass.
- S4 is mandatory even when S7 (upgrade to a proof) proof is planned.

## 4. Rollback

Wrong property → S2 (freeze specifications and contracts); model too narrow/broad → S3; instrumentation error → repair instrumentation and rerecord within S4; real code failure → S6 (find and repair bugs). Carry the trace hash, event index and source anchor. Use [a rollback ticket](../../templates/rollback.md).

## Concrete Examples

A PD slot model can pass its configured safety checks while rejecting a real slot-reuse trace. Diagnose the first divergence before using that model to justify a repair.
