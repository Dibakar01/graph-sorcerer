---
name: graph-sorcerer
description: Use when work may be wide enough to run as a fleet of parallel agents rather than one — any "do X for each of these N things" job across many files, sources or angles. Also use when asked to design a graph, a dynamic workflow or an ultracode run; to parallelize, fan out or speed up a slow chain of agent steps; or to judge whether a job should be a graph at all. Reach for it before spawning parallel agents by hand — establishing that a job is NOT wide enough is one of its main jobs.
---

# Graph Sorcerer

Designs agent graphs — and refuses to build one when the work isn't actually wide.

## The core principle

A graph buys **breadth**. It never buys judgment. Fanning out serial work multiplies the bill and
adds failure modes for zero speedup.

So the most valuable output of this skill is often **"this shouldn't be a graph."** Say that
plainly when it's true — someone who wanted a fleet and got a correct answer about why a loop is
better has been served well.

**Never launch a fleet without an explicit go-ahead.** Fleets spend real money in the background,
and the failure mode is silent: a graph pointed at the wrong job produces a confident, complete,
expensive, wrong report. Design first, show the design, iterate, then run only when told to.

## Phase 1 — Gate

Do this before designing anything.

**Run the fake-edge test.** List the jobs. For each arrow between two jobs, ask one question:
*does this step actually need the result of the one before it?* If yes, the edge is real. If no,
there is no edge — the wait is an artifact of the order someone typed things in, and those jobs
can run at once.

**Then check the four skip conditions.** Any one of these means don't build a graph:

- The task is small or isolated — one function, one bug. Coordination costs more than it saves.
- The user wants to approve every step. A graph's whole point is running wide without them.
- Nobody knows what they're looking for yet. Exploratory work wants one steerable agent.
- The steps genuinely depend on each other.

**State the honest speedup.** Elapsed time is the critical path, not the sum of the work — but
the serial fraction sets the floor (this is Amdahl's law). If 40% of the work is genuinely
sequential, infinite agents still only buy ~2.5×. Estimate the fraction and say the number out
loud. **If the realistic win is under about 2×, say the graph isn't worth it** and explain why.

If the gate fails, stop here. Report what the work actually is — usually a loop — and what to do
instead. Don't soften it into "we could build a small graph anyway."

## Phase 2 — Design

The shape is almost always the **diamond**: fan out → reduce → verify → synthesize. Fan out for
breadth, reduce in plain code (no model, no tokens), verify on fresh context, synthesize once at
the end. Cheap models on mechanical nodes, the strong model where judgment lives.

Four things every design needs, each earning its place:

**Node contracts.** One bounded job, defined input, defined output. A node returning free text is
one only a human can read. Use `agent({schema})` so the shape is enforced, not hoped for.

**Verifier independence.** The agent that did the work never checks it — models miss most of their
own mistakes. A verifier that read the worker's chat inherited its framing and errors: that is the
first measurement quoted twice, not a second one. **Fresh context**, and *different* questions
(correct? current? source real?) — identical skeptics share a prior and catch the same things.

**A hard cap.** Claude Code's "Dynamic workflow size" guideline is **advisory, not enforced**
(default: under 15 agents), so write a real cap into the spec. Scope the first run small.

**An anchor.** A graph comparing its own outputs to its own outputs measures consistency and
reports it as truth; more nodes make it more confident, not more correct. Name one input set
outside the system — a test that ran, a number the graph can't rewrite. **No anchor, don't widen.**

Full contracts and the orchestration skeleton are in `references/patterns.md`.

Also design for the three ways graphs break — context collapse, false independence, silent node
death. Fixes are in `references/patterns.md`.

## Phase 3 — Present, then iterate

Show the design and stop. Include:

- the topology (what fans out, what merges, where the real edges are)
- the node and verifier contracts
- the cap, the anchor, and any human gates
- a cost and time estimate, with the serial fraction stated

Then ask what to change, and iterate until the user confirms it's right. Treat their edits as
binding. This loop is the point — it's cheap to redraw a graph and expensive to run the wrong one.

## Phase 4 — Run, only on a green signal

Launch only after an explicit go-ahead. Then honour the design: the cap is real, the gates stop
the run and wait, and every merge counts its inputs against what it expected so a dead node
can't slip into a report that looks complete. A run that worked is worth keeping — offer to save
it (see `references/patterns.md`).

## Matching the house style

A fleet of agents will otherwise produce a dozen different house styles in one codebase. Before
fanning out, read the nearest `CLAUDE.md` / `AGENTS.md` and pass the relevant conventions into
**every** worker prompt — not just the lead. Also carry across anything the user has set for this
run: cap, model tiers, human gates, the anchor, and whether workers need isolated worktrees.

## When you catch yourself thinking this

| Thought | What's actually true |
|---|---|
| "These are probably independent" | Probably isn't a test. Two nodes writing one file or hitting one rate-limited API share a hidden edge. Check the resources, not just the prompts. |
| "The verifier can reuse the worker's context, it's cheaper" | Then it isn't a verifier. Shared context means shared errors — you've paid extra for agreement. |
| "I'll add a cap once I see how it goes" | The size guideline won't stop it. Uncapped is how a run becomes a four-figure invoice in the background. |
| "More nodes will make it more accurate" | More nodes make it more consistent. Without an anchor, consistency and confident wrongness look identical. |
| "They asked for a fleet, so build a fleet" | They asked for the outcome. If a loop gets it faster and cheaper, that's the better answer — say so. |
| "It's mostly serial but a graph is more impressive" | It's slower and pricier. Amdahl doesn't negotiate. |

## Reference

`references/patterns.md` — verified Claude Code mechanics (`agent({schema})`,
`isolation: "worktree"`, size guideline, sandbox limits), the diamond skeleton, the three
failure-mode fixes, and paste-ready spec templates. Read it once a graph is warranted; the gate
doesn't need it.
