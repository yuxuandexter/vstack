---
name: vstack-s8-package-maintain
description: "Package verified scope into a project-local five-step regression skill and keep it aligned with future code revisions."
---

# S8 — Package and maintain regression

Read [Layer 0: shared operating rules](../../docs/layer-0.md), this stage and the assigned input bundle.

## Role and write scope

One author per case skill; the coordinator owns gold and accepts independent reruns. Only the coordinator integrates changes.

## 1. Input

The accepted artifacts from S4 (calibrate and align traces), S5 (attack model fidelity and gate strength), S6 (find and repair bugs) and S7 (upgrade to a proof), according to the S1 (split obligations and select methods) route: models, configurations, traces, mutations, expectations and hashes.

## 2. Procedure

1. Fill [verify-skill](../../templates/verify-skill.md): coverage, synchronized revision, assumptions/bounds, one command, expected results and triage rules.
2. Use five steps: pins → drift → model check → trace regression → mutation regression. Target a small repeatable regression, but measure and report its actual duration.
3. Break protected logic in a source copy: the check must turn red. Restore it: green with the baseline. Record prediction versus actual, including failed predictions.
4. On upstream changes, run unchanged first, inspect every change to covered state (including moved/new writers), then classify no effect, model drift, intentional promise change or possible code bug.
5. Update models from source, not from desired verdicts. Recalibrate after alignment changes and update the sync revision only after alignment succeeds. Keep verification and product-fix changes distinct.
6. For a deductive claim, retain the frozen proof tree, tool/settings hashes, raw logs and [full acceptance evidence](../../docs/gates.md#deductive-proof-acceptance). Publish the root statement and premises verbatim. Changes to definitions or the invariant require affected-closure checks and renewed fresh acceptance; the five-step regression alone does not renew a proof claim.

## 3. Exit checks

- Break/restore drill has both results; independent clean rerun matches every expected row.
- The skill distinguishes baseline reproduction from a required-fix check and explains stale frozen trace evidence.
- Maintenance ends as in sync, updated, bug or blocked. A blocked run does not advance the synchronized revision.
- Deductive claims identify the proved design model; describe the implementation as validated against it. An independent reviewer must complete the proof gates before a machine-proof claim.

## 4. Rollback

Model drift → S3 (build an anchored model) then S4 regression; intentional promise change → S2 (freeze specifications and contracts); code violation → S6; unavailable tools/alignment → BLOCKED without moving the sync commit. Use [a rollback ticket](../../templates/rollback.md).

## Concrete Examples

The [PD example](../../examples/verify-pd-kv-slot/SKILL.md) describes pins, drift, model checks, trace checks and mutations. Frozen trace checks can stay unchanged after a live source mutation; drift and source-derived model variants must detect that change.
