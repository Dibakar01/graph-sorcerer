<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/mascot-dark.svg">
  <img alt="" src="assets/mascot-light.svg" width="104">
</picture>

# Graph Sorcerer

**Designs agent graphs — and tells you when you don't need one.**

[![License: MIT](https://img.shields.io/badge/License-MIT-d92819?style=flat-square)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-2.1%2B-1a1211?style=flat-square)](https://claude.com/claude-code)
[![Refuses narrow work](https://img.shields.io/badge/refuses_narrow_work-by_design-7a6a68?style=flat-square)](#when-it-refuses-you)

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/hero-dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="assets/hero-light.svg">
  <img alt="Your job enters a gate that runs the fake-edge test. No edge, and it builds a capped, verified graph. Real edge, and it refuses and says why." src="assets/hero-light.svg" width="100%">
</picture>

</div>

---

## Install

```bash
claude plugin marketplace add Dibakar01/graph-sorcerer
claude plugin install graph-sorcerer@graph-sorcerer
```

Then just ask, in your own words. No keyword to memorise.

<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/cost-dark.svg">
  <img alt="Cost to leave installed: about 130 tokens always-on, drawn to scale as a hairline against a 200k context window — 0.065%." src="assets/cost-light.svg" width="100%">
</picture>
</div>

---

## It never starts a fleet on its own

<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/loop-dark.svg">
  <img alt="You describe a job. The gate either stops there and says why, or produces a design. You review it and loop back with changes; only your green signal starts the run." src="assets/loop-light.svg" width="100%">
</picture>
</div>

A fleet spends real money in the background, and the failure is quiet — a graph pointed at the
wrong job returns a confident, complete, expensive, **wrong** report. So you approve the design
before anything spawns.

---

## The four checks

Any model parallelises when asked. These are what get skipped under time pressure.

| | Check | What it catches |
|:--:|---|---|
| ◇ | **Fake-edge test** | If B never reads A's output, that arrow is just the order you typed things in. Delete it, run both at once. |
| ⏱ | **Honest speedup** | Elapsed time is the critical path, not the sum — but the serial fraction sets the floor. 40% serial means infinite agents still only buy ~2.5×. You get the number, not a vibe. |
| ⊘ | **Verifier independence** | A checker that read the worker's chat inherited its errors. That's not a second measurement, it's the first one quoted twice. Fresh context, three *different* questions. |
| ⚓ | **An anchor** | A graph comparing its own outputs to its own outputs measures consistency and reports it as truth. More nodes = more confident, not more correct. |

Plus a **hard cap** — Claude Code's workflow-size setting is [advisory, not enforced](#notes-on-accuracy).

---

## Same request. Two outcomes.

<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/ab-dark.svg">
  <img alt="The same request, two outcomes. Without the plugin eight agents spawn on work whose every step needs the last one's result. With it, the gate finds no independent pair and declines, leaving one agent." src="assets/ab-light.svg" width="100%">
</picture>
</div>

None of this is exotic. It is the set of checks that get skipped at 5pm on a Thursday — which is
exactly when skipping them costs you.

| | Without it | With it |
|---|---|---|
| **Serial work, asked in parallel** | You get a fleet. The model complies — that is what it is for. | It declines, and names the work a loop. |
| **Verification** | The worker's own context checks its own output. It agrees, because it already believed it. | A separate node on fresh context, asked three *different* questions. |
| **Agent count** | The workflow-size setting is a hint to the model, not a limit. Nothing stops a runaway fan-out. | A hard cap, written into the spec itself. |
| **A node dies mid-run** | Silent. The report reads complete, on partial data. | Every merge counts its inputs and flags the gap. |

It does not make your agents smarter. It stops you paying for breadth that was never there — and
stops a confident wrong answer reaching you dressed as a finished report.

---

## What it's worth

<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/roi-dark.svg">
  <img alt="Break-even in tokens. One wrongly fanned-out job of eight agents at fifteen thousand tokens each wastes 105,000 tokens. The plugin costs 130 tokens per session — a hairline on the same scale — so one prevented fan-out covers roughly 800 sessions." src="assets/roi-light.svg" width="100%">
</picture>
</div>

Fan **N** agents at work whose every step needs the last one's result and you get one agent's worth
of progress. **(N−1) × T** tokens are simply gone. Against a measured always-on cost of 130 tokens
per session, one correctly-declined fan-out covers a lot of sessions:

| Agents wrongly spawned | Tokens per agent | Wasted | Sessions of plugin cost covered |
|---|---|---|---|
| 4 | 8,000 | 24,000 | ~185 |
| 8 | 15,000 | 105,000 | ~800 |
| 8 | 25,000 | 175,000 | ~1,350 |
| 16 | 25,000 | 375,000 | ~2,900 |

**Read that honestly.** It is arithmetic, not a benchmark. The 130 tokens is measured; **N** and
**T** are your numbers. And the figure it does *not* contain is the one that would turn this into
an ROI claim — **how often the model fans out when it shouldn't.** That is a behavioural rate, it
needs measuring rather than reasoning, and `bench/run.sh` is the harness for it.

So the claim here is deliberately narrow: **if** it saves you from one unnecessary fleet, it has
paid for its own presence for months. Whether it does is what the evals are for.

---

## The shape it builds

```mermaid
flowchart LR
    S([split]) --> W1[worker] & W2[worker] & W3[worker] & W4[worker]
    W1 & W2 & W3 & W4 --> R[reduce<br/><i>plain code, 0 tokens</i>]
    R --> V[verify<br/><i>fresh context</i>]
    V --> M([synthesize])
```

Fan out for breadth · reduce in code · verify on fresh context · synthesize once.
Cheap models on mechanical nodes, the strong one where judgment lives.

---

## When it refuses you

<table>
<tr><td width="50%">

**✗ It declines**

> Fix the failing test in `auth.spec.ts`

Read → diagnose → fix → re-run. Every step needs the last one's result. One agent is faster and cheaper.

</td><td width="50%">

**✓ It builds**

> Audit all 40 route files for missing auth

40 independent jobs, 2 real edges. Proposes a diamond, caps the first run at 20, flags any file that doesn't return.

</td></tr>
</table>

It also refuses when you want to approve every step, when you don't yet know what you're looking
for, or when the steps genuinely depend on each other.

> **The tell:** if no two jobs are independent, there's no graph to build. It's a loop — and a loop is fine.

---

## Test it yourself

```bash
claude plugin eval graph-sorcerer --max-cost-usd 2
```

Runs each case **with** the plugin and again **without**, then reports the delta. The interesting
one is `refuses-narrow-work`: the baseline arm happily fans out a one-file bug fix; the plugin arm
declines.

---

<details>
<summary><b>Notes on accuracy</b> — three things the source article gets wrong</summary>

<br>

The idea comes from Anatoli Kopadze's [*Graph Engineering explained*](https://x.com/AnatoliKopadze/status/2080668775796314331) — a genuinely good explainer, worth reading. Every mechanism here was re-verified against the CLI rather than taken from the article, which surfaced three corrections:

| | Correction |
|---|---|
| **The keyword moved** | The article says start your prompt with `workflow`. That was renamed to `ultracode` in CLI 2.1.160 — the changelog states plainly that "the word 'workflow' no longer triggers a run" — and a later build narrowed it further. Plain-language requests have worked throughout. **Don't rely on a magic word.** |
| **The size guideline is advisory** | `/config` → "Dynamic workflow size" is a hint to the model (default: under 15 agents), **not an enforced limit.** Write a real cap into the spec. |
| **Two patterns are now native** | `agent({schema})` gives an enforced node contract; `isolation: "worktree"` gives each worker its own git worktree. No need to hand-roll either. |

The underlying scheduling ideas are far older than the framing suggests — dataflow graphs, MapReduce, DAG orchestrators, and critical-path analysis from 1950s operations research. What's genuinely new is that the nodes are **non-deterministic and expensive**, which is why verification and cost caps have to be structural rather than best practice.

</details>

<details>
<summary><b>What's in the box</b></summary>

<br>

```
skills/graph-sorcerer/
├── SKILL.md              the gate and the judgment
└── references/
    └── patterns.md       verified CLI mechanics · the diamond skeleton
                          · fixes for the 3 ways graphs break
                          · 6 paste-ready spec templates
evals/
├── refuses-narrow-work/  the case that matters
├── builds-wide-work/
└── partial-mixed-work/
```

`patterns.md` loads only once a graph is warranted — the gate doesn't need it.

</details>

<details>
<summary><b>The three ways graphs break</b></summary>

<br>

| Failure | Fix |
|---|---|
| **Context collapse** — 1,000 outputs into one final step blows the window before synthesis starts | Layer the fan-in: batch, summarize each batch, combine the summaries |
| **False independence** — two nodes write the same file or hit the same rate-limited API. A hidden edge | Give every worker its own worktree; audit shared *resources*, not just shared data |
| **Silent node death** — one dead node among 200 slips into a report that looks complete | Every merge counts its inputs against what it expected, and flags the gap |

</details>

---

---

## Credits & source

The thinking behind this plugin is **Anatoli Kopadze's**, from
[*Graph Engineering explained: what it is, when to use it and when not to*](https://x.com/AnatoliKopadze/status/2080668775796314331)
(24 July 2026). This is an implementation of his ideas, not a substitute for reading them — the
article is genuinely good and worth your time.

| Principle used here | Source |
|---|---|
| **Node / edge** — a node is one bounded job; an edge only counts when data actually passes along it | §2 |
| **The fake-edge test** — *does this step need the result of the one before it?* | §3 |
| **The diamond** — fan out → reduce → verify → synthesize | §5 |
| **The checker needs a fresh context**, and three *different* questions beat three identical ones | §6 |
| **The three failure modes** — context collapse, false independence, silent node death | §7 |
| **When not to build a graph** — the four skip conditions | §8 |
| **Anchors** — topology doesn't buy truth; name something the graph can't rewrite | §9 |
| **The bill is real** — the Bun rewrite at ~$165k / 11 days / 64 agents | §12 |

### What this repo added

| | |
|---|---|
| **Amdahl's law, named** | The article asserts the serial-fraction ceiling but never names or draws it. The gate now states the honest speedup as a number, and says so when it's under ~2×. |
| **Three CLI corrections** | Every mechanism was re-verified against the Claude Code changelog rather than taken from the article — see [Notes on accuracy](#notes-on-accuracy). |
| **The refusal, enforced** | The article describes when *not* to build a graph. This makes declining the default behaviour rather than advice you have to remember at 5pm on a Thursday. |
| **Eval cases** | Three, including one the plugin is supposed to *fail* to build. |

Older than any of it: dataflow graphs, MapReduce, DAG orchestrators, and critical-path analysis
from 1950s operations research. Kopadze concedes this in §1 and is right to — a pattern that has
run critical systems for thirty years is exactly what you want underneath your work.

**On IP.** The code and prose in this repository are MIT-licensed and written here. The ideas are
cited above and belong to their author; nothing in this repo reproduces the article's text or its
diagrams.

<div align="center">

**Requirements:** Claude Code 2.1+ · **Licence:** [MIT](LICENSE)

<sub>Proposing a change to the gate? Include an eval case that fails without it and passes with it —<br>a rule nobody can measure is a rule that quietly rots.</sub>

</div>
