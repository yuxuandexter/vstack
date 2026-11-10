---
name: vstack-s2-spec
description: "Derive observable contracts, calibrate both legal and illegal behavior, then freeze owner-approved specifications and configurations."
---

# S2 — Freeze specifications and contracts

Read [Layer 0: shared operating rules](../../docs/layer-0.md), this stage and the assigned input bundle.

## Role and write scope

The coordinator writes top-level promises and pins; the property owner approves them. Workers supply anchored evidence and staged proposals.

## 1. Input

The approved S1 (split obligations and select methods) design and obligation matrix.

## 2. Procedure

1. Write a short, reviewable statement of observable consequences and explicit non-goals. Use [contract discovery](../../docs/contracts.md) and [the card template](../../templates/contract-card.md).
2. Require top-level decomposition plus historical bugs or consumer requirements; ledger/protocol families complete one-step induction. Scan all seven consumer-precondition categories.
3. Calibrate negative examples, legal edge behavior, witnesses and minimum scale before freezing. S4 (calibrate and align traces) repeats calibration on the complete model.
4. Expose environment assumptions and ambiguous semantics as sourced switches. State which interpretation each claim requires.
5. The coordinator pins the approved text, ideal model and configurations deciding what is checked. Before deductive proof, also freeze the complete inductive invariant and root statement, including explicit parameter ranges and configuration premises. Check content hashes before every counted run.

## 3. Exit checks

- Approval, provenance, consumer scan and calibration records are complete: partly manual.
- `scripts/check_spec_pins.sh ROOT PINS` succeeds: mechanical byte comparison.
- The frozen ideal rejects its approved bad-implementation suite while controls pass: project-specific execution, not automatic approval.

## 4. Rollback

Use a rollback ticket for an overly strong property or missing assumption. Stage the change, compare full regressions, obtain a new property-owner decision, then let the coordinator independently rerun and re-pin. Changing the inductive invariant also requires rechecking Init and every affected preservation cell in the same proof tree, followed by fresh acceptance; old proof results do not carry over. Use [a rollback ticket](../../templates/rollback.md).

## Concrete Examples

For a PD KV slot, a posted write must not simply disappear from the ideal model. Freeze that promise and the configurations that check it; revise them only through the property owner and coordinator.
