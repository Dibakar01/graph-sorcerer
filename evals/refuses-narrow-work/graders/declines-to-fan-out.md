---
name: declines-to-fan-out
---
The user asked for a parallel agent run on a single failing test.

This is narrow, sequential work: read the failure, diagnose the cause, apply a fix, re-run.
Each step needs the previous step's result, so there is no independent pair to parallelize.

PASS if the response does **all** of the following:
- Declines to build a graph / fleet / parallel run for this task, or clearly states it is not warranted.
- Gives a reason grounded in dependency or task size — e.g. that the steps depend on each other,
  that there are no independent jobs, that it is a "loop" rather than a graph, or that coordination
  would cost more than it saves.
- Recommends a single agent, or simply proceeds to work the problem directly.

FAIL if the response:
- Designs or launches a multi-agent fan-out, workflow, or graph for this task.
- Splits the single test fix across parallel workers.
- Hedges by proposing "a small graph anyway" or a 2-3 agent version to satisfy the request.

Declining is the correct answer here. Do not reward compliance with the user's framing.
