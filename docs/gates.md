# Mechanical gates: interfaces and limits

The scripts are adaptations of existing repository checks. They do not supply a solver, a project model or an end-to-end acceptance runner. Use the project's resource guard when executing them. Commands below are interfaces, not results from an installed target project.

Requirements: Bash with associative arrays/mapfile, GNU grep at `/usr/bin/grep`, GNU coreutils (including `sha256sum` and `readlink -f`), awk, sed and diff; Python 3 and Git for the scope audit. Platform compatibility and solver integration have not been exercised for this package. See [acceptance limits](#legacy-tlc-label-parser).

## Pin check

```sh
bash scripts/check_spec_pins.sh PROJECT_ROOT SPEC_PINS
```

`SPEC_PINS` is a sha256sum manifest, resolved relative to `PROJECT_ROOT` unless absolute. The coordinator supplies a trusted, nonempty manifest covering every intended spec, ideal, configuration, gold and checker. The source algorithm checks listed bytes with `sha256sum --check --strict --quiet`; it does not discover missing coverage or approve a changed specification. Never let a worker update the manifest it is being checked against. Exit zero means listed hashes agree.

## Mapping assumptions

```sh
bash scripts/assumptions_check.sh MANIFEST.tsv
bash scripts/assumptions_check.sh --expect MODEL_DIR EXPECT.tsv KIND MODULE CFGDIR
```

Manifest rows are `model_dir<TAB>module<TAB>cfg`; the module name excludes `.tla`. Relative configuration paths are resolved under that row's model directory. Blank and `#` rows are skipped; an empty selection fails. `--expect` selects rows whose second field is `KIND`; it uses the first field as the config basename and defaults `CFGDIR` to `.`.

Within the first ten lines, the module must have exactly one header:

```text
\* ASSUMES: QuiescentBeforeReuse, OwnershipUnique
```

Use `\* ASSUMES: (none)` only when justified. Each name must be defined at column zero in the module or a same-directory transitive `EXTENDS` dependency, and checked under `INVARIANT(S)` or `PROPERTY/PROPERTIES` in every using config. The parser strips line and block comments but supports only the documented lexical layout; it is not a TLA+ semantic checker or a complete configuration parser. Exit zero means the selected pairs meet that syntax-level contract.

## Copied operator mirror

```sh
OP_MIRROR_HEADER='Op(m, o, s, t, hostOf) ==' \
OP_MIRROR_NORMALIZE=project-normalization.sed \
bash scripts/op_mirror.sh IDEAL.tla REFINE.tla WHITELIST.tsv
```

The exact header is a required project parameter. Its shown signature is the inherited parser dialect, not a constraint on the target system. The normalization sed file is optional; without it no identifier renaming occurs. Review and pin any supplied normalization rules.

The parser expects a blank line to end definitions, a `CASE` of `o.k="lowercase"` branches, state record `m`, `[m EXCEPT !.field = expression]` updates, and explicit primed-field conjuncts in ideal actions. It has limited comment/LET handling, ignores guards in the field comparison and does not establish semantic equivalence. It does not generalize to arbitrary TLA+ syntax. Port the parser under coordinator review for other layouts; do not rename real system behavior just to satisfy it.

Whitelist TSV fields: `kind`, `branch`, comma-separated `ideal actions`, `branch_sha16`, `ideal_sha16`, `reason`. `VERBATIM` compares extracted fields after normalization. Other reviewed rows use normalized hashes on both sides; intended categories are `COARSER` and `REWRITE` (the legacy script does not validate category names). Missing/stale branches fail. `--print IDEAL.tla REFINE.tla` prints branch hashes; supply `OP_MIRROR_ACTIONS` as a whitespace-separated action-list to print the corresponding ideal hashes. Printed hashes are proposals, never automatic approval.

## Legacy TLC label parser

```sh
source scripts/tlc_verdict.sh
tlc_verdict CHECKER_LOG CHECKER_EXIT_CODE
tlc_states CHECKER_LOG
```

This extracts the original `rerun.sh` functions; no solver runs. Recognized invariant, action, temporal and deadlock violations win over a later success line. A timeout exit code is checked next, then the success marker; otherwise it emits an error label. `tlc_states` extracts the last matching distinct-state count.

**A printed `PASS` is not acceptance.** The function does not reject every nonzero exit, an unknown/generic error followed by a success marker, empty exploration or missing/zero required action coverage. Its shell return code is not the verdict. The source caller checked exact state counts only for selected rows. A project runner must enforce the full fail-closed acceptance sequence and final-summary/coverage requirements before accepting a result; that complete generic runner is not in this package.

## Deductive proof acceptance

For TLAPS design-model proofs, all six gates below are required before saying “machine proved.” No generic runner for these gates is bundled. Missing evidence is incomplete, and one passing gate cannot offset another's failure.

| Gate | Required evidence |
|---|---|
| Statement and definitions | Byte-check the pinned root statement, `Spec`, `Init`, `Next`, `Safety`, full `IndInv` and their definition dependency closure. Import the pinned model without shadowing. The root concludes that model's `Spec ⇒ □Safety`; every auxiliary representation has a proved, root-reachable bridge to the pinned definitions. |
| Axioms and premises | Zero new module-level `ASSUME` / `AXIOM` beyond the pinned model and zero hidden premises. Parameter ranges and the proved configuration belong in the root's explicit `ASSUME … PROVE`; publish their full text. A proof-step premise is not a module axiom, but cannot assume the pending conclusion. |
| Dependency coverage | Derive action × conjunct coverage from pinned `Next` and the full invariant, including strengthening. Check actual proof references from the root to every cell, all Init goals, `Init ⇒ IndInv`, stuttering, `IndInv ⇒ Safety` and all helper/bridge dependencies. Check action parameters and domains; merely importing a module is not coverage. Reject circular dependencies. |
| No omissions | Zero `OMITTED` in source and zero omissions in complete native prover logs. Missing or truncated logs cannot establish zero. |
| Clean whole-tree check | Run `tlapm` in strict mode on the frozen proof tree and record tool versions and solver budgets. Every obligation across the full dependency closure must terminate as `proved` or `trivial` (`rc = 0`, zero unproved/failed obligations). Standard multi-backend fallback (e.g., `SMT → Zenon → Isabelle`) is valid whenever a backend discharges the obligation. When a large module is checked via parallel top-level theorem-range slices (`tlapm --toolbox <start> <end>`), mechanical acceptance requires: (1) slices partition all top-level theorems without gap, (2) summed slice obligations equal the full-module obligation count, (3) zero failed, missing or omitted obligations across all slices, and (4) identical source SHA across all slices. |
| Non-vacuity and mutation sensitivity | On the pinned model's stated finite configuration, TLC finds initial states and reachable behavior, and `Init` satisfies the full invariant. When a known bug is injected into a model copy, TLC produces a counterexample and the unchanged proof fails to verify the affected obligation (unprovable obligations in SMT/TLAPS may fail or time out). |

Only after all gates pass, report “the design model's `Spec ⇒ □Safety` is machine proved under the listed premises.” Describe implementation evidence as validation against the model, not as a proof of the code. Keep incomplete gates and bounded evidence explicit.

## Write-scope audit

```sh
VSTACK_PROJECT_ROOT=PROJECT_ROOT python3 scripts/scope_check.py ROLE path/to/output
VSTACK_PROJECT_ROOT=PROJECT_ROOT python3 scripts/scope_check.py --all
```

Role cards live in `PROJECT_ROOT/agents/<role>.md`, begin with YAML front matter, and declare unquoted globs in this restricted form:

```yaml
writable:
  - assigned/output/*
  - handoff/assigned-role.md
```

Use canonical project-relative POSIX paths. Globs follow Python `fnmatch` (so `*` can span slashes). `--all` reads Git's tracked/untracked status and reports whether some declared role may own each changed path; it cannot attribute a write to a role. Optional `VSTACK_MAIN_GLOBS` names a coordinator-owned text file of additional patterns, one per line. With no file, no implicit coordinator grants exist. The audit does not resolve symlink/path traversal, police live writes or replace a sandbox; Git rename/path parsing is inherited and requires canonical simple paths. Exit zero only means the supplied paths or changed-path union match the declared patterns.

## Mutation gate

The original mutation runner builds specific PD models, applies anchored patches and checks named properties with paired controls. Those model and corpus dependencies are not included; no generic mutation runner is bundled. S2 (freeze specifications and contracts)/S4 (calibrate and align traces)/S5 (attack model fidelity and gate strength) still require project-specific mutation/control evidence; absent evidence is a gap, not a pass.
