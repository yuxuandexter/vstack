# Find contracts before freezing them

One card per contract family. Describe observable consequences, not the implementation that happens to exist.

| Source | Required work |
|---|---|
| Top-level decomposition | Trace the contract to an approved system promise. Always required. |
| Historical bugs | Negate an evidenced bad state; record what the fix actually establishes at pinned buggy/fixed revisions. |
| Consumer requirements | At every handoff, work backward from unchecked consumer preconditions to producer Requires/Ensures. |
| One-step induction | On a small ledger or protocol model, classify each counterexample as reachable (operation defect) or unreachable (missing strengthening fact). |

Top-level decomposition is mandatory, together with historical bugs or consumer requirements. Ledger/protocol contracts must complete one-step induction before freezing, including all three induction checks. Computation descriptors instead need differential checks against the actual functions.

## Consumer scan: seven categories

At every cross-component, cross-thread, device or stream boundary check:

- indices and offsets: who establishes bounds;
- value ranges: legal identifiers, positions and lengths;
- overwrite points: all writers and their ordering;
- execution context: producing/consuming stream or thread, event placement;
- lifetime: how long an asynchronously consumed allocation remains alive;
- aliases and views: whether retained data is a copy or a mutable view;
- initial values: whether anything can read a location before valid initialization.

## Two-sided calibration

A contract needs a negative execution that violates it, positive reachability witnesses and edge-legal behaviors that it accepts, and the minimum scale that exposes its failure. Explain why smaller bounds miss the failure. Run both the minimum and one increment. Add normal and fault-injected real traces when available.

The contract-level calibration belongs in S2 (freeze specifications and contracts) before freezing. S4 (calibrate and align traces) repeats it in the full model and adds runtime alignment; do not postpone all calibration to S4.

Freeze only when every in-scope model has cards, every boundary has the consumer scan, ledger/protocol induction has no unclassified counterexamples, calibration and minimum bounds are recorded, and the property owner has approved the promises. The coordinator pins specs **and configurations**. Before deductive proof, pin the full inductive invariant and root statement too. Put parameter ranges and the proved configuration in explicit root `ASSUME … PROVE` premises and publish them verbatim; do not add module-level axioms. The card checklist is partly manual; the hash comparison is mechanical.

Discover strengthening before proof begins. A later invariant change requires new pins, Init and affected preservation checks in the same module tree, and renewed [proof acceptance](gates.md#deductive-proof-acceptance). Auxiliary definitions stay in the proof layer with proved bridges to the pinned model.
