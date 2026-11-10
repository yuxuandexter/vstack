# Choose by the property

Use two independent axes: the mathematical difficulty determines the representation; scale and the assurance objective determine the verification tier. Execution hardware is not the decision boundary.

| Difficulty | Approach |
|---|---|
| Small owned state with severe consequences if corrupted | Isolate a small state-machine model; check its contracts and keep environment assumptions and the implementation-alignment gap explicit. |
| Asynchronous interleavings, messages, timeouts, faults, hardware interpretations | TLA+ state machine, TLC exploration, trace alignment and reproduction; escalate the same model when needed. |
| Stateless dispatch over input dimensions or modes | Z3 constraints over anchored descriptors; compare descriptors to real functions. |
| Algebraic correctness and floating-point reduction order | Separate formula correctness, reduction structure and dispatch; check composition. Handwritten descriptors remain an abstraction boundary. |

For each obligation answer: what makes it difficult; whether scale is bounded; who owns the code and how often it changes; how severe failure is. Map the obligation to ledger, protocol, computation, alignment and fallback guarantees. State the weakest link in every public claim.

## State-machine tiers

1. **Explicit state checking:** TLC explores configured finite domains and gives counterexamples. Appropriate for evolving code and small hardware-fixed participant counts; include deadlock, liveness and alternative hardware readings where relevant.
2. **Symbolic checking:** Apalache/Z3 tests larger symbolic domains or falsifies inductive invariants with a single transition. A bounded symbolic verdict is not an unbounded proof.
3. **Deductive proof:** TLAPS for the existing TLA+ design model. A claim beyond finite deployment bounds needs a corresponding unbounded theorem. Do not rewrite a calibrated TLA+ model in another language merely to upgrade tools.

Confirm tool availability before committing to a tier. Do not copy historical installation status from a case into a new project. S4 (calibrate and align traces) and S5 (attack model fidelity and gate strength) still apply before a proof upgrade.

## Safety and IndInv are different artifacts

The property owner approves **Safety**, the observable promise; the coordinator freezes it. Discover **IndInv** with named strengthening facts before deductive proof, then have the coordinator hash-pin the complete invariant and specification. A forward run from Init does not establish inductiveness. Later strengthening requires new pins, Init and affected preservation checks in the same proof tree, and renewed full acceptance.

In a scratch wrapper, check:

```text
Initiation:  Init => IndInv
Consecution: IndInv /\ Next => IndInv'
Implication: IndInv => Safety
```

The source workflow uses these Apalache commands (configure the tool version and finite domains for your project):

```sh
apalache-mc check --config=APTarget.cfg --init=Init --inv=IndInv --length=0 APTarget.tla
apalache-mc check --config=APTarget.cfg --init=IndInv --inv=IndInv --length=1 APTarget.tla
apalache-mc check --config=APTarget.cfg --init=IndInv --inv=Safety --length=0 APTarget.tla
```

For every consecution counterexample, decide whether the predecessor is reachable. If reachable, investigate a bug; if unreachable, record the named conjunct that excludes it, then rerun all three checks. Add type annotations only in scratch copies when the wrapper needs them. Do not import those wrappers into the delivered proof.

Decompose preservation by **action × invariant conjunct** in one shared module tree. Follow [S7 — Upgrade to a proof](../skills/vstack-s7-prove/SKILL.md) for projections, action effects, frame facts and local contexts. Proof-layer abstractions need proved bridges to the pinned definitions; do not replace the calibrated model. Never cite a TLC or Apalache success as justification for a TLAPS proof step.

Require all [proof acceptance gates](gates.md#deductive-proof-acceptance), including the original target, explicit root premises, complete dependencies and a clean whole-tree check. This package supplies guidance and spec pinning, **not a generic proof-integrity checker**.
