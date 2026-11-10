# Layer 0: every role, every stage

These rules apply to verification work on concurrent, distributed and systems software. A project supplies its own resource guard, source snapshots and writable paths. The plugin does not grant execution or publication permissions.

| Rule | Requirement | Enforcement and limits |
|---|---|---|
| Output ownership | One task, one output owner. Write only the assigned paths. Workers do not spawn workers or commit. | `scripts/scope_check.py` audits declared paths; it is not a filesystem sandbox and does not prove who wrote a file. The coordinator alone integrates changes. |
| Resource and access limits | Put expensive work under the project's CPU/memory/time guard. Register roles before dispatch. Use script files instead of inline Python. External source access is read-only and rate-limited. | Resource guards and read-only access wrappers are environment-provided; none is bundled. |
| Handoff continuity | Create a handoff at kickoff; refresh it at least every 15 minutes. Record conclusions on disk, not merely documents read. Keep bulk logs in assigned scratch space. | Documented protocol; no timer enforcement. Report paths and concise conclusions to the coordinator. |
| Fail-closed acceptance | Fail closed: violations first; then timeout, unknown or errors; then empty state exploration or missing/zero required action coverage; only then accept success. Read final summaries, not progress lines. | The bundled `tlc_verdict.sh` is the **legacy log classifier**, not a complete acceptance gate. Its missing checks are listed in [gate limits](gates.md). A legacy PASS alone never satisfies this rule. |
| Independent reruns | The coordinator independently reruns submitted artifacts in a clean directory, using trusted tools and expectations. | Project runners must rebuild results from pinned inputs; worker-produced result files are not evidence of an independent run. |
| Frozen-input ownership | Only the coordinator owns approved specs, configuration, gold expectations, verdict scripts and pin manifests. Workers read them. | `scripts/check_spec_pins.sh` checks bytes against a trusted manifest. It does not establish authorship, approval or semantic adequacy. Protect the manifest itself outside worker write scopes. |

## Stage contract

Every stage has an owner and writable paths, then four cells: **input**, **procedure**, **exit checks**, **rollback**. Input must have passed the upstream exit and carry hashes. Each exit check identifies its enforcement as mechanical, partly manual, documented-only or missing. A handoff verdict is PASS, ISSUES or BLOCKED; gaps do not count as PASS.

Workers read this document, their own stage skill and the inputs named in the task. Do not rederive upstream conclusions or silently edit upstream files.

## Per-property routing

Select stages for each property in S1 (split obligations and select methods). S4 (calibrate and align traces) calibration/trace alignment and S5 (attack model fidelity and gate strength) red team are mandatory. S6 (find and repair bugs) may be skipped only for newly written code whose property is headed for deductive proof. Proof is not required for every hardware-bounded problem; report the chosen tier and its limits.

## One-way rollback tickets

A downstream worker may create a ticket to the responsible upstream stage; it cannot mutate that stage's frozen output. Use `templates/rollback.md`: identify source and destination stages, artifact hashes, counterexample or trace evidence, proposed change and approver. After an accepted repair, re-enter the affected stage and rerun its downstream checks.

Changes to promises, ideal models or frozen configurations require an explicit property-owner decision; the coordinator applies the change, independently checks it and re-pins. Gold and verdict changes require coordinator review. Never weaken the model or property merely to make a run green.

## Handoff and process record

Record completed paths, current work, remaining work, evidence locations and the next concrete action. Preserve an append-only UTC worklog and decision trail. For long checks, use isolated process groups under the project launcher, record their IDs, and verify termination before handing back; never kill other roles' processes by name.
