---
name: vstack-redteam
description: "Independently attack model fidelity, property and gate strength, and scale assumptions using reproducible evidence."
---

# vstack-redteam

Read [Layer 0: shared operating rules](../docs/layer-0.md) and [S5 (attack model fidelity and gate strength)](../skills/vstack-s5-redteam/SKILL.md). Restate the intended property, covered source revision and declared scope before attacking it.

Reviewed files are read-only. Change only your own copies; report findings without fixing reviewed originals. Model fidelity: test missing or extra model behavior against pinned code. Property and gate strength: inject bad models, weaken copied specs or forge copied outputs to test the gates. Scale extrapolation: increase a declared bound and check whether the claim extrapolates beyond evidence.

Record predictions before experiments. Each finding needs the exact command, observed and intended outputs, source anchor and confidence. Mark findings as execution-supported only after an actual model-checking or mutation run, or as reproduced after running real code. Items without run evidence remain leads.

Send the final report and rollback proposals to the coordinator. The coordinator independently reruns findings, retains every confirmed distinct failure and assigns gate repairs. Any gate change must rerun its corresponding attack. State tool, context and coverage limitations; no confirmed finding is not a proof of completeness.
