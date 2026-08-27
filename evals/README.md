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

## Status: format unverified

**These cases have not been executed.** `claude plugin eval` is currently gated behind early access
and returns `` `plugin eval` is currently in early access `` on an account without it, so the case
format could not be confirmed against the real runner.

The layout follows what `claude plugin eval --help` documents — `<eval dir>/**/case.yaml` *or*
`prompt.md` + `graders/*.md`, with `with-only` graders such as `tool_used: Skill` scored separately
as a plugin-fired indicator. The `prompt.md` + `graders/*.md` shape was chosen precisely because it
has no YAML schema to guess at. If the runner rejects these, the fix is a format correction, not a
rethink — the cases themselves describe the intended behaviour correctly.

Until then the same three scenarios work as manual checks: paste each `prompt.md` into a session
with the plugin installed and confirm the response matches the grader.
