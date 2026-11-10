# Review a checker result

Before running, timestamp the sentence you expect to report. Read constants, state/action constraints, symmetry, view, properties, worker count and limits. Do not interpret a process exit code as a correctness theorem.

Ask seven questions:

1. Which state-space dimension or resource limits the result?
2. Are compared models using the same constants, constraints and properties?
3. Does the state count make sense, or did Init/constraints prune the behavior?
4. Were there errors, warnings, memory failures, timeouts or exploration caps?
5. Is the result reproducible? For measurements used to choose between alternatives, alternate at least five runs per side and report median/range.
6. Does the range include the smallest known trigger?
7. Were fault actions actually explored? Supply coverage and reachability witnesses.

Report **violation found**, **passed within stated bounds**, **no measurable difference** (measurement only), or **INCONCLUSIVE**. Missing evidence for errors/coverage, zero required action coverage, unmatched comparison configurations or an unexplained bottleneck prevents acceptance. Unfinished work, unknown and timeout are never success.

Read the final generated/distinct-state summary, not a progress line. Violation counts from multi-worker searches can vary with scheduling; do not silently require exact counts for those rows. The source rerun table distinguishes exact checks from informational state counts.

A killed mutant must violate the expected property **and follow the expected mechanism**. Compare the historic switch combination to the actual pinned revision. A repair component outside the abstraction's reach is unverified, not proved unnecessary. Pair each positive run with its reachability witness under the same constants.
