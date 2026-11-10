---
name: vstack-s7-prove
description: "Prove pinned design-model properties in one shared TLAPS module tree after invariant discovery, calibration and trace alignment."
---

# S7 — Upgrade to a proof

Read [Layer 0: shared operating rules](../../docs/layer-0.md), this stage and the assigned input bundle.

## Role and write scope

Proof workers own assigned proof modules and auxiliary lemmas in one shared module tree. The coordinator owns theorem statements, invariant pins and accepted tool/settings manifests.

## 1. Input

The calibrated model selected in S1 (split obligations and select methods), after S4 (calibrate and align traces)/S5 (attack model fidelity and gate strength), with no unclassified model/code mismatch. Complete [invariant discovery](../../docs/method-selection.md) first; pin the specification, full inductive invariant, parameter ranges and proved configuration before deductive proof starts.

## 2. Procedure

1. Import the pinned model directly. The root's conclusion is exactly its `Spec ⇒ □Safety`; parameter ranges and configuration appear as explicit root `ASSUME … PROVE` premises. Add no module-level `ASSUME` or `AXIOM`. Keep necessary auxiliary definitions in the proof layer, with machine-checked bridges to pinned definitions; never substitute a model rewritten for easier proofs.
2. Use one module tree and shared definitions from the start. Assign action × conjunct cells within that tree; do not prove independent model copies and rename/merge them at the end. Include all strengthening conjuncts in the cell inventory.
3. Prove projection lemmas `IndInv ⇒ Conjunct`; unfold the full invariant only at this interface. Each preservation cell receives just the old-state conjuncts it needs, including necessary type/domain facts.
4. Expand an action once to establish effect equations and frame facts (unchanged fields). Subsequent cells use those facts. Prove shared effects as family lemmas while retaining every action-to-family bridge; keep large local definitions hidden after proving their interfaces.
5. Split remaining goals by quantified object, updated versus unchanged object, or old versus new set member. Close Init, every preservation cell, stuttering and `IndInv ⇒ Safety`; the root only composes proved results. Never cite bounded model-checking success as a proof premise.
6. Control facts and definitions separately: `BY ONLY` retains already usable definitions. Pair it with `HIDE DEF` and keep `USE DEF` inside the smallest subproof scope. A renamed large expression still expands unless hidden.
7. Diagnose single-threaded proof-manager preprocessing (`normalize_expand`) separately from backend search (`--threads` only parallelizes backend solvers) and whole-module wall-clock interruption. Reduce context expansion for internal slowdowns; for large modules, parallelize preprocessing across OS processes by slicing along top-level theorem boundaries with `tlapm --toolbox <start_line> <end_line> --threads 1` so source SHA and module dependencies remain unchanged. When a single obligation fails, isolate and debug its line range with `--toolbox` before rerunning the module. If a module hits its wall-clock limit with zero failed obligations, rerun with range slicing or a doubled wall-clock budget rather than editing the proof.
8. Enforce at most one in-flight check per module across the cluster, schedule downstream closure refreshes along true `EXTENDS` dependencies rather than coarse batch barriers, and retain fingerprint caches across reruns to skip redundant frontend expansion. Recheck the affected dependency closure through the root after every integration; an invariant change invalidates affected evidence and requires re-pinning and full re-verification.

This structure follows the [TLAPS guide](https://lamport.azurewebsites.net/tla/tla2-guide.pdf), [practical hints](https://proofs.tlapl.us/doc/web/content/Documentation/Tutorial/Practical_hints.html), [tactics](https://proofs.tlapl.us/doc/web/content/Documentation/Tutorial/Tactics.html), [MongoRaftReconfig proof organization](https://arxiv.org/html/2109.11987v2) and [TLAPS-Bench theorem-range slicing](https://arxiv.org/abs/2611.02849). It is not a measured performance guarantee.

## 3. Exit checks

- All [proof acceptance gates](../../docs/gates.md#deductive-proof-acceptance) pass on the frozen tree, including bridges and strengthening. Cached leaves or a green root alone are insufficient.
- Report the root statement and its premises verbatim. Finite-domain induction retains its bounds; an arbitrary-scale claim requires the corresponding quantification.
- The proved object is the design model. Describe implementation evidence as validation against that model, with its alignment limits.
- Timeout, unknown, omitted, interrupted or inconclusive work is incomplete. No generic proof-integrity/closure checker is bundled.

## 4. Rollback

A reachable induction counterexample → S6 (find and repair bugs). A missing premise or needed invariant change → S2 (freeze specifications and contracts) for review and new pins; do not inherit the old acceptance result. If strengthening fails, revisit S3 (build an anchored model). Use [a rollback ticket](../../templates/rollback.md).

## Concrete Examples

For PD KV slot release, prove the design-model promise only after checking real interleavings and trace conformance. A theorem about an atomic model step does not establish atomicity in the implementation.
