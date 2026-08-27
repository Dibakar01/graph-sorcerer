---
name: splits-parallel-from-serial
---
This job is half wide and half narrow. The research angles are independent of each other and can
run at once. Everything after the merge — outline, draft, edit — is serial, because the post
depends on the merged research.

PASS if the response does **both**:
- Parallelizes only the research portion, and keeps the writing sequential (or explicitly says the
  writing cannot be meaningfully parallelized).
- Is honest about the limited speedup — e.g. names the serial fraction, says the gain applies only
  to the research half, or gives a bounded estimate rather than implying a large uniform speedup.

FAIL if it:
- Proposes fanning the whole job out uniformly, including the writing.
- Implies the speedup scales with agent count without acknowledging the serial tail.
