# Mechanical gates: interfaces and limits

These adapted repository checks provide no solver, project model or end-to-end acceptance runner. Use the project's resource guard. The commands below define interfaces; they are not results from an installed project.

Requirements: Bash with associative arrays and `mapfile`, GNU grep at `/usr/bin/grep`, GNU coreutils with `sha256sum` and `readlink -f`; awk, sed and diff; Python 3 and Git for the scope audit. Platform compatibility and solver integration have not been exercised for this package. See [acceptance limits](#legacy-tlc-label-parser).

## Pin check

```sh
bash scripts/check_spec_pins.sh PROJECT_ROOT SPEC_PINS
```

Resolve the sha256sum manifest `SPEC_PINS` relative to `PROJECT_ROOT` unless absolute. The coordinator supplies a trusted, nonempty manifest covering all intended specs, ideals, configurations, gold and checkers. `sha256sum --check --strict --quiet` compares listed bytes; it cannot detect missing coverage or approve specification changes. Never let workers update their reference manifest. Exit zero means listed hashes agree.

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

Supply the exact header for your project. The shown signature belongs to the inherited parser dialect; it does not constrain the target system. Normalization uses an optional sed file. Without it, no identifiers are renamed. Review and pin any supplied rules.

The parser expects a blank line to end definitions, a `CASE` of `o.k="lowercase"` branches, state record `m`, `[m EXCEPT !.field = expression]` updates, and explicit primed-field conjuncts in ideal actions. It has limited handling of comments and LET, ignores guards when comparing fields and does not establish semantic equivalence. It does not generalize to arbitrary TLA+ syntax. Port the parser under coordinator review for other layouts; do not rename real system behavior just to satisfy it.

Whitelist TSV fields: `kind`, `branch`, comma-separated `ideal actions`, `branch_sha16`, `ideal_sha16`, `reason`. `VERBATIM` compares extracted fields after normalization. Other reviewed rows use normalized hashes on both sides. Use the intended categories `COARSER` and `REWRITE`; the legacy script does not validate category names. Missing or stale branches fail. `--print IDEAL.tla REFINE.tla` prints branch hashes; supply `OP_MIRROR_ACTIONS` as a whitespace-separated action-list to print the corresponding ideal hashes. Printed hashes are proposals, never automatic approval.

## Legacy TLC label parser

```sh
source scripts/tlc_verdict.sh
tlc_verdict CHECKER_LOG CHECKER_EXIT_CODE
tlc_states CHECKER_LOG
```

These functions come from `rerun.sh`; they run no solver. Recognized invariant, action, temporal and deadlock violations take precedence over later success. Next come the timeout exit code, then the success marker; otherwise the parser emits an error label. `tlc_states` extracts the last matching distinct-state count.

**A printed `PASS` is not acceptance.** The function can accept nonzero exits, unknown or generic errors followed by success, empty exploration, and missing or zero required action coverage. Its shell return code is not the verdict. The source caller checked exact state counts only for selected rows. Require a project runner that enforces the full fail-closed sequence, final summaries and coverage. No complete generic runner is bundled.

## Deductive proof acceptance

For TLAPS design-model proofs, require the five mathematical gates below before saying “machine proved.” No generic runner for these gates is bundled. Missing evidence is incomplete; one passing gate cannot offset another's failure.

| Gate | Required evidence |
|---|---|
| Statement and definitions | Byte-check the pinned root statement, `Spec`, `Init`, `Next`, `Safety`, full `IndInv` and their definition dependency closure. Import the pinned model without shadowing. The root concludes that model's `Spec ⇒ □Safety`; every auxiliary representation has a proved, root-reachable bridge to the pinned definitions. |
| Axioms and premises | Require zero new module-level `ASSUME` or `AXIOM` beyond the pinned model and zero hidden premises. Put approved parameter ranges and the proved configuration in the root's explicit `ASSUME … PROVE`; publish their full text. A proof-step premise is not a module axiom, but cannot assume the pending conclusion. |
| Dependency coverage | Derive every action and invariant-conjunct cell from pinned `Next` and the full invariant, including strengthening. Check actual proof references from the root to every cell, all Init goals, `Init ⇒ IndInv`, stuttering, `IndInv ⇒ Safety` and all delivered helpers and bridges. Check action parameters and domains. Prove explicitly when approved premises disable an action; never silently drop it. Reject circular dependencies and orphan proofs. Merely importing a module is not coverage. |
| No omissions | Require zero `OMITTED` in source and zero omissions in complete native prover logs throughout the closure. Missing or truncated logs cannot establish zero. |
| Complete strict results | Retain one complete strict result for every module in the root's full dependency closure. Require `rc = 0`, a final `proved` or `trivial` for every obligation and zero final failed, unknown, timeout or missing obligations. Reconcile results with native complete counts. Use a whole-module run or a slice certificate satisfying the conditions below. |

Keep S4 calibration, non-vacuity and S5 attacks as stage prerequisites; retain their evidence when the pinned model and configuration are unchanged. Report additional mutation diagnostics separately from proof completion. A timeout alone never proves a mutant false; pair proof non-discharge with a matching model-checker counterexample and identify whether it starts at Init or an arbitrary inductive state.

Record elapsed time, peak memory, budgets and repeat runs for diagnosis and reproducibility. Do not impose a half-budget finish, a per-module memory threshold or a second same-source run as a mathematical gate. Enforce resource protection through the [run policy](run-review.md#proof-runs); a killed or unfinished run supplies no complete result. Judge backend chains by each obligation's final status: an intermediate timeout or failure followed by a proof is valid fallback.

### Result identity and reuse

Bind each accepted result to the checked module bytes, its complete transitive `EXTENDS` and `INSTANCE` closure, external libraries and toolchain. Record resolved import paths, source and tool hashes, commands, settings, raw logs and cache provenance. Recompute identities against the assembled tree and verdicts from raw obligation events; do not trust a summary boolean alone. Results from different rounds may be combined when these identities match.

Lock passing results whose source and closure remain unchanged. If a dependency changes only its proof body, preserve downstream source and refresh its result with persistent fingerprints after its own changed dependencies pass. If exported definitions or statements change, review the affected proof contracts before rechecking; a refresh alone cannot establish compatibility. Recheck affected dependencies through the root, then repeat the static gates on the assembled tree.

Accept native fingerprint hits from strict runs under the same frozen toolchain and libraries. A fingerprint hashes the complete sequent, including premises, goal and proof directives; it is not just a source-file hash. Keep provenance for the successful cache entries. Do not require a cache-disabled rerun of an otherwise valid result.

### Slice certificates

For parallel theorem ranges using `tlapm --toolbox <start> <end>`, require all five conditions mechanically:

1. Cut at top-level theorem or lemma boundaries in the complete module context. The ranges must cover every proof without gaps or duplicate obligations.
2. Match the sum of slice `obligationsnumber` counts to the whole-module native count under the same source, closure and toolchain. Reconcile obligation IDs and source locations to detect duplicates and omissions; equal totals alone are insufficient.
3. Require strict mode without `--lax`, `rc = 0` for every slice, all final obligations `proved` or `trivial`, zero final failed, unknown, timeout or missing obligations, and zero native `OMITTED`.
4. Match every slice's module source, full import closure and toolchain hashes. Record each slice's actual settings and cache provenance.
5. Retain slice paths, ranges, counts, hashes and raw logs in the module certificate. Have an independent reviewer recompute conditions 1–4 from source and raw logs, independently of the author's certificate builder.

A complete slice certificate satisfies the module check; no additional whole-module proof run is required. A count-only run supplies a denominator, never a proof. A diagnostic line check without complete coverage remains partial.

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

Supply canonical project-relative POSIX paths. Globs follow Python `fnmatch`; `*` can span slashes. `--all` reads Git's tracked and untracked status and checks whether a declared role may own each changed path. It cannot identify the writer. Optional `VSTACK_MAIN_GLOBS` names a coordinator-owned file with one additional pattern per line; no file means no implicit coordinator grants. The audit neither resolves symlinks or traversal nor polices live writes. It does not replace a sandbox. The inherited Git rename and path parser requires canonical simple paths. Exit zero means only that the supplied paths or changed-path union match declared patterns.

## Mutation gate

The original mutation runner builds specific PD models, applies anchored patches and checks named properties with paired controls. Those model and corpus dependencies are not included; no generic mutation runner is bundled. S2, S4 and S5 still require project-specific mutation and control evidence; absent evidence is a gap, not a pass.
