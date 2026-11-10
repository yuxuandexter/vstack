---
name: vstack-s5-redteam
description: "Audit calibrated verification through independent attacks on fidelity, property/gate strength and scope extrapolation."
---

# S5 — Attack model fidelity and gate strength

Read [Layer 0: shared operating rules](../../docs/layer-0.md), this stage and the assigned input bundle.

## Role and write scope

Red team reads reviewed artifacts and mutates only its own copies. It reports findings; the coordinator verifies them and assigns fixes.

## 1. Input

The complete S4 (calibrate and align traces) bundle and its scope statement, with hashes.

## 2. Procedure

1. Assign independent attack directions when delegation is available and authorized: model fidelity (missing/extra behaviors), property and gate strength (bad models, weak specs, forged output), scale extrapolation (one larger configuration).
2. For each finding record exact command, actual output, intended output, source anchors and confidence. Mark findings as execution-supported when a run supports them, or as reproduced on real code. Items without run evidence remain leads.
3. The coordinator reruns findings and merges common causes. Keep every independently confirmed failure, not just one reviewer's list.
4. Turn a successful bypass into a mechanical check where feasible and record its ablation. Rerun the corresponding attack after gate changes.

## 3. Exit checks

- Every execution-supported or reproduced finding is resolved or has a rollback ticket; coordinator review is manual.
- Update the attack-to-enforcer table with evidence and remaining gaps.
- Gate changes have their attack regressions. The source has an ASSUMES selftest; a complete generic attack suite is not bundled.
- S5 is mandatory; a model author's success report is insufficient.

## 4. Rollback

Fidelity → S3 (build an anchored model); weak ideal → S2 (freeze specifications and contracts) with property-owner approval; weak verdict/gate → coordinator-owned gate repair. Never modify the reviewed originals from the red-team role. Use [a rollback ticket](../../templates/rollback.md).

## Concrete Examples

For PD KV slot ownership, attack a copied ideal action, an unchecked mapping assumption and an implementation that silently discards a posted write. These exercise mirror, ASSUMES and specification-mutation checks.
