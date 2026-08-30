# Patterns and verified mechanics

Read this once the gate in `SKILL.md` has passed and a graph is actually warranted.

## Contents

- [What Claude Code actually provides](#what-claude-code-actually-provides)
- [Triggering a dynamic workflow](#triggering-a-dynamic-workflow)
- [The diamond skeleton](#the-diamond-skeleton)
- [The three failure modes and their fixes](#the-three-failure-modes-and-their-fixes)
- [Contracts](#contracts)
- [Saving a run](#saving-a-run)
- [Spec templates](#spec-templates)

---

## What Claude Code actually provides

Verified against the installed CLI changelog rather than blog posts — a lot of writing about
agent graphs describes mechanics that have since moved.

| Mechanic | What's true | Why it matters |
|---|---|---|
| **Dynamic workflows** | Native. Claude writes an orchestration script and spawns a coordinated fleet to run it. `/workflows` lists saved ones; a detail view shows the agent grid and per-phase progress. | Coordination happens in code, not conversation, so passing results between agents doesn't re-spend the session's context. |
| **`agent({schema})`** | Structured output with enforced validation; a subagent aborts after 5 consecutive schema-validation failures rather than looping. | This *is* the node contract. Use it instead of asking for well-formed text and hoping. |
| **`isolation: "worktree"`** | Supported on subagents and workflow agents — each works in its own temporary git worktree. | This *is* the fix for false independence. Use it whenever two nodes could touch the same file. |
| **Size guideline** | `/config` → "Dynamic workflow size" (small/medium/large/unrestricted), also settable as the `workflowSizeGuideline` key in any settings file. Default is medium: aim for fewer than 15 agents. **Explicitly advisory, not an enforced cap.** | Nothing stops a runaway fan-out for you. The cap has to be in the spec. |
| **Workflow sandbox** | Workflow scripts run sandboxed; dynamic `import()` can't escape it. Validation rejects non-determinism such as `Date.now()` / `Math.random()` in script logic. | Keep orchestration deterministic. Push anything time- or randomness-dependent into a node's prompt, not the script. |
| **Prefix caching** | Sibling agents sharing a prompt prefix are staggered so later ones read the cached prefix instead of re-paying for it (`CLAUDE_CODE_WORKFLOW_PREFIX_STAGGER_MS=0` disables). | Keep the shared part of worker prompts identical and put the varying part last — it's cheaper. |
| **Saving** | User scope `~/.claude/workflows/`, project scope `.claude/workflows/`; the nearest `.claude/` wins on a name collision. | A good run becomes a named command instead of a prompt someone has to remember. |
| **Telemetry** | Workflow-spawned agents emit `workflow.run_id` and `workflow.name` OpenTelemetry attributes. | A run's cost and activity can be reconstructed afterwards. |

## Triggering a dynamic workflow

**Don't depend on a magic keyword.** It has moved more than once:

- The trigger keyword was renamed from `workflow` to **`ultracode`** (CLI 2.1.160), and that entry
  states plainly that the word "workflow" no longer triggers a run.
- A later change (2.1.178) describes the keyword firing on explicit phrases such as
  `"run a workflow"` or `"workflow:"`, rather than on any mention of the word.
- There is also a "Workflow keyword trigger" setting in `/config` that can switch keyword
  triggering off entirely.

Widely-shared guides still say *"start your prompt with the word workflow"*. On a current CLI
that is unreliable.

Both changelog entries agree on the durable part: **asking for one explicitly, in plain words,
works.** So state the request directly — *"Run this as a dynamic workflow: ..."* — and treat any
keyword as a shortcut, never the mechanism. If a run doesn't fan out, check `/config` rather than
guessing at magic words.

## The diamond skeleton

Fan out → reduce → verify → synthesize. Adapt the angles and prompts; the skeleton is the same
behind a market scan, a code review, and a research report.

```js
const angles = [
  "pricing vs the top 3 competitors",
  "what buyers complain about in reviews",
  "the feature gaps in the category",
  "where the market moves in the next 12 months",
];

// FAN OUT — one worker per angle, all at once
const raw = await parallel(
  angles.map(a => () => agent({
    task: `research: ${a}. every claim needs a source url + date.`,
    schema: Finding,        // enforced contract, not free text
    model: "cheap",         // mechanical node → cheap model
  }))
);

// REDUCE — plain code. No model, no tokens, no chance of invention.
const findings = dedupeBySource(raw.flat().filter(Boolean));

// VERIFY — a fresh skeptic per finding, trying to kill it
const survivors = await parallel(
  findings.map(f => () => agent({
    task: "try to disprove this. return keep | drop + why.",
    input: f,               // the finding only — never the worker's chat
    freshContext: true,
    model: "strong",        // judgment node → strong model
  }))
).then(v => findings.filter((_, i) => v[i].verdict === "keep"));

// SYNTHESIZE — one agent writes the answer from what survived
return agent({
  task: "one report, ranked by confidence, sources attached.",
  input: survivors,
  model: "strong",
});
```

Note what the reduce step is: ordinary code. Compression is the cheapest thing in the graph and
the one place a model adds risk rather than value.

## The three failure modes and their fixes

### 1. Context collapse

Fan out a thousand nodes, pour all thousand outputs into one final step, and the context window
is gone before synthesis starts.

```js
// layered fan-in — summarize in batches, then combine the summaries
const batches = chunk(results, 40);
const summaries = await parallel(
  batches.map(b => () => agent({ task: "summarize this batch", input: b }))
);
return agent({ task: "write the answer from the summaries", input: summaries });
// the final step reads ~25 summaries, not 1,000 raw outputs
```

### 2. False independence

Two nodes look independent because their prompts never mention each other, but they write the
same file or hit the same rate-limited API. That's a hidden edge, and it corrupts results rather
than failing loudly.

```js
await parallel(files.map(f => () => agent({
  task: `refactor ${f}`,
  isolation: "worktree",   // each agent gets its own temporary git worktree
})));
// rule: any two nodes writing the same file need an edge, not parallelism
```

Audit for shared *resources*, not just shared data.

### 3. Silent node failure

In a chain, one failure stops everything — obvious. In a graph, one dead node among two hundred
slips into a report that looks complete.

```js
const results = (await parallel(jobs)).filter(Boolean);
if (results.length < jobs.length) {
  flag(`WARNING: ${jobs.length - results.length} of ${jobs.length} nodes returned nothing`);
}
// never synthesize on a partial set and call the report complete
```

## Contracts

**Node contract** — what makes a node wire-able:

```
▸ NODE CONTRACT
JOB:     research one competitor's pricing (one job, nothing else)
IN:      { competitor: "name", url: "https://..." }   ← passed in, never assumed
OUT:     { price: number, plan: string, source: url, date: "YYYY-MM-DD" }
SCHEMA:  enforced via agent({schema}); free text is rejected and retried
WHY:     a defined output lets the next node read this one without a human
         in the middle. That is what makes it wire-able.
```

**Verifier contract** — what makes a check a real check:

```
▸ VERIFIER NODE
INPUT:    one finding from a worker (the finding only, never the worker's chat)
CONTEXT:  fresh and empty — it has not seen the work it is judging
CHECKS:   three skeptics in parallel, each with a DIFFERENT question
  1. is it correct?      → does the claim actually hold up
  2. is it current?      → is the source recent, not stale
  3. is the source real? → does the link resolve to the claim it's cited for
PASS:     keep the finding only if a majority let it live
FAIL:     drop it before it reaches the final answer
```

Three identical skeptics share a prior and catch the same things; three different questions is
what makes the check independent.

## Saving a run

A run that worked is worth keeping. Save it to `~/.claude/workflows/` (user scope) or
`.claude/workflows/` (project scope) and it becomes a named command you can launch directly
instead of re-describing the graph. `/workflows` lists what is saved; when names collide, the
`.claude/` nearest the working directory wins.

Save after a run has proven itself, not before — a saved bad graph is a bad graph you will now
run by reflex.

## Spec templates

Swap the bracketed parts. State the request explicitly rather than relying on a keyword — see
[Triggering a dynamic workflow](#triggering-a-dynamic-workflow). Keep a human as the last yes
before anything ships.

**First run — start scoped.** Prove the shape and see the cost before widening.

```
Run this as a dynamic workflow.

▸ GRAPH SPEC
GOAL: audit every route file under src/routes/ for missing auth checks

FAN OUT:    one agent per file, in parallel
VERIFY:     an independent checker on each finding, fresh context
CAP:        20 files on this first run
ON FAIL:    flag any file that doesn't return, never skip it silently
REPORT:     one merged list of the routes missing auth
```

**Research desk.** Breadth with a skeptic on every finding.

```
Run this as a dynamic workflow.

▸ GRAPH SPEC
GOAL: decision-grade research on [your question]

FAN OUT:      5 distinct angles, one researcher per angle, in parallel
RULE:         every finding needs a source link and a date
VERIFY:       a skeptic attacks each finding and tries to disprove it; drop what fails
MERGE:        survivors into one report ranked by confidence
ANCHOR:       [a source the graph cannot rewrite]
SAVE:         research-report.md, then show me the top findings
HUMAN GATE:   ask me before changing anything after that
```

**Content draft.** Parallel research, serial writing — the honest shape.

```
Run this as a dynamic workflow.

▸ GRAPH SPEC
GOAL: one ranking-ready draft for [topic]

PARALLEL JOBS (run at once):
  1. what the current top-ranking pages cover
  2. the real questions people ask about this topic
  3. what those top pages skip
MERGE:        the three into an outline, then write a full draft
VERIFY:       a fact-checker that flags every claim without a source
SAVE:         drafts/ with the flagged claims listed at the top
HUMAN GATE:   never publish anything
```

**Launch kit.** Two fan-outs with a human gate between them.

```
Run this as a dynamic workflow.

▸ GRAPH SPEC
GOAL: full launch kit for [product], aimed at [audience]

PARALLEL JOBS (research, at once):
  1. profile the buyer and the exact words they use
  2. map where these buyers spend time online
  3. collect how competitors pitch them
MERGE:        a one-page positioning doc
HUMAN GATE:   pause and show me the positioning doc before writing
PARALLEL JOBS (writing, from that doc):
  1. landing page copy
  2. a week of launch posts
  3. a set of outreach messages
VERIFY:       a checker compares every asset to the positioning doc, flags anything off
SAVE:         launch-kit/ — then ask me before changing anything
```

**Refactor sweep.** Isolation is mandatory here — the workers all write code.

```
Run this as a dynamic workflow.

▸ GRAPH SPEC
GOAL: find every function over 100 lines and propose a refactor for each

FAN OUT:    one agent per file, in parallel, each in its own worktree
VERIFY:     an independent checker on each proposed refactor, fresh context
DEDUPE:     proposals against everything already seen
CAP:        50 files on this first run
ANCHOR:     the test suite must still pass — proposals that break it are dropped
REPORT:     how many files came back, so nothing fails silently
```

**Discovery loop.** For work whose size isn't known until you're in it, where finding one thing
reveals three more. The stop condition and the cap are what keep it from running away.

```
Run this as a dynamic workflow.

▸ GRAPH SPEC
GOAL: hunt this repo for [security issues / broken error handling / dead code]

FAN OUT:    run finders in parallel
DEDUPE:     check each new find against everything already seen
VERIFY:     an independent checker on the survivors
LOOP:       keep going until two rounds in a row find nothing new, then stop
CAP:        a hard limit on total agents so it can't run away
REPORT:     final list ranked by severity
```

Run one scoped, watch what it costs, then widen. When a run is good, save it — every one of
these becomes a single command launched by name.
