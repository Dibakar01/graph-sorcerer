# Contributing

Issues and pull requests are welcome.

## The one rule that matters

**A change to the gate needs an eval case that fails without it and passes with it.**

The gate is the whole product — it decides whether a job becomes a fleet of agents or gets turned
down. A rule nobody can measure is a rule that quietly rots, and a gate that drifts toward "yes"
is indistinguishable from having no gate at all.

So if you are changing when the skill refuses, widen `evals/` in the same PR:

```
evals/your-case/
├── prompt.md            what the user asks
└── graders/
    └── your-check.md    what a correct response must and must not do
```

Run them with `claude plugin eval graph-sorcerer --max-cost-usd 2`. The runner executes each case
with the plugin and again without it, so you can see whether your change actually did anything.

If a case passes in **both** arms it is non-discriminating — it measures nothing and needs
rewriting.

> `claude plugin eval` is currently in early access. If you can't run it, say so in the PR and
> describe the manual check you did instead. That's fine; silently skipping it isn't.

## Style

- Prose in `SKILL.md` is paid on every invocation, so it earns its place or it goes. Detail belongs
  in `references/patterns.md`, which loads only once a graph is warranted.
- The frontmatter `description` is paid in **every session**, whether or not the skill fires. Treat
  additions there as expensive.
- Explain *why* a rule exists. A model that understands the reason generalises; one following a
  bare instruction does not.

## Reporting a mechanism that has changed

Claude Code moves fast, and this plugin documents CLI behaviour. If something here is stale, please
open an issue with the CLI version and the changelog line — that is exactly how the three
corrections in the README were found.
