---
name: vstack-s1-select
description: "Choose verification tools and assurance tiers per property using state, scale, ownership and consequences."
---

# S1 — Split obligations and select methods

Read [Layer 0: shared operating rules](../../docs/layer-0.md), this stage and the assigned input bundle.

## Role and write scope

The coordinator owns the design and obligation matrix; the property owner approves the scope.

## 1. Input

S0 (reconstruct and separate the system) control/data/boundary maps and verified anchors, with hashes.

## 2. Procedure

1. Use [method selection](../../docs/method-selection.md). Split by property, not module or processor type.
2. For every obligation answer: difficulty, finite/unbounded scale, code ownership/change rate, failure severity.
3. Map ledger, protocol, computation, alignment and fallback guarantees. Identify which row supports each promise and which is weakest.
4. Record tier and rationale, check tool availability, and record any skipped stages per property. S4 (calibrate and align traces)/S5 (attack model fidelity and gate strength) cannot be skipped; S6 (find and repair bugs) only for new code destined for deductive proof.

## 3. Exit checks

- Every hazardous transition and consumer precondition has an obligation: manual review.
- No unexplained holes in the guarantee matrix: no generic checker is bundled.
- Scope and design approval are recorded; actual guarantee wording follows the weakest supporting evidence.

## 4. Rollback

Missing maps return to S0. If no small clean proof core exists, propose the smallest interface separation before proving tangled orchestration code. Use [a rollback ticket](../../templates/rollback.md).

## Concrete Examples

PD KV slot reuse is a state-machine obligation: finite exploration finds interleavings; an arbitrary-scale claim needs a corresponding design-model theorem plus explicit model/code alignment.
