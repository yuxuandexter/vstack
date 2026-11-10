---
name: vstack-s6-hunt-fix
description: "Search within a trace-validated model scope, classify counterexamples and validate fixes in both the model and real code."
---

# S6 — Find and repair bugs

Read [Layer 0: shared operating rules](../../docs/layer-0.md), this stage and the assigned input bundle.

## Role and write scope

Workers write new hunt/fix artifacts; approved properties and prior patches stay read-only.

## 1. Input

The model reviewed in S5 (attack model fidelity and gate strength), restricted to the scope actually trace-validated.

## 2. Procedure

1. Write timestamped predictions for each bug family before running. Enable known repairs so shallow known failures do not hide deeper ones; record every switch.
2. Parameterize hardcoded constants first. At the old values, require matching verdicts/state counts; only then increase one dimension.
3. Classify each counterexample as model defect, known bug or new candidate. Attach code steps and a minimal trigger.
4. Reproduce progressively: source-level reasoning, real control-code replay with stubs, controlled timing on the real system. State the actual level and any injected behavior.
5. Compare repair candidates in the model: original failure gone, other safety/liveness preserved, removal restores failure. Then change code and check unpatched N/N versus patched 0/N under the declared trigger.

## 3. Exit checks

- Every candidate is triaged and its reproduction scope recorded.
- Repair controls and full regression agree with frozen expectations.
- A known-bug baseline matching expectations is not the same as a repaired implementation passing; require the fixed-state contract when validating a repair.

## 4. Rollback

Unrealizable counterexample → S3 (build an anchored model), or S2 (freeze specifications and contracts) if the promise/assumption is wrong. A repair requiring a weaker promise returns to S2 for approval. Use [a rollback ticket](../../templates/rollback.md).

## Concrete Examples

For PD KV slot reuse, compare repairs that preserve in-flight accounting across reuse. Check both safe release and eventual release, then remove each repair component to test its role.
