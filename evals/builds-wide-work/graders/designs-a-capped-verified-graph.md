---
name: designs-a-capped-verified-graph
---
This is genuinely wide work: 40 files, each auditable without reading any other file's result.
A fan-out is the right call here.

PASS if the response proposes a parallel design AND includes **all three** of:
- A **hard cap** or explicitly scoped first run (e.g. "start with 20 files"), rather than an
  uncapped fan-out across all 40.
- An **independent verifier** on findings, described as having fresh/clean/separate context —
  not the same agent or context that produced the finding.
- A **fan-in count / completeness check**, so a worker that returns nothing is flagged rather than
  silently dropped from the report.

Also PASS the response if it presents the design for approval before running it.

FAIL if it proposes an uncapped fan-out, omits verification, or has workers grade their own output.

Do NOT require git worktree isolation here — the task is explicitly read-only, so nothing can be
overwritten. Proposing worktree isolation anyway is unnecessary but not disqualifying; requiring
it would be wrong.
