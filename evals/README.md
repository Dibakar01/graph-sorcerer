# Eval cases

Three cases for `claude plugin eval`, one per verdict the gate can reach.

| Case | Correct outcome |
|---|---|
| `refuses-narrow-work` | **Declines.** A single failing test is sequential — no independent pair, so no graph. |
| `builds-wide-work` | **Builds.** 40 independent file audits, with a cap, a fresh-context verifier and a fan-in count. |
| `partial-mixed-work` | **Splits.** Research fans out, writing stays serial, and the speedup is stated honestly. |

`refuses-narrow-work` is the one that earns its keep. Without the plugin a model will cheerfully
fan out a one-file bug fix because the user asked for parallelism; with it, it should decline. If a
case passes in both the with-plugin and without-plugin arms it is non-discriminating and the grader
needs rewriting — a grader that always passes measures nothing.

## Running them

```bash
claude plugin eval graph-sorcerer --max-cost-usd 2
```

`--ablation with-without` is the default once the plugin resolves, so the no-plugin baseline arm
comes for free and the output reports the delta between the two.

## Requirements

`claude plugin eval` is currently in early access.

The cases follow the layout its `--help` documents — `<eval dir>/**/case.yaml`, or `prompt.md` plus
`graders/*.md`, with `with-only` graders such as `tool_used: Skill` scored separately as a
plugin-fired indicator. The `prompt.md` + `graders/*.md` shape is used here because it has no YAML
schema to match.

Until then the same three scenarios work as manual checks: paste each `prompt.md` into a session
with the plugin installed and confirm the response matches the grader.
