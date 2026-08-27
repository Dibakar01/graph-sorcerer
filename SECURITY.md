# Security

## What this plugin ships

Markdown only — a skill definition, a reference document, and eval cases. **No executable code, no
hooks, no MCP servers, no network calls, no install scripts.** `claude plugin details` will confirm
the inventory.

The realistic risk is therefore not code execution but *instruction content*: the skill shapes how
an agent plans work, and it can propose commands for you to run. Read what it proposes, as you
would any generated plan.

## Reporting

Open a [security advisory](https://github.com/Dibakar01/graph-sorcerer/security/advisories/new),
or a regular issue if it is low severity. Please don't include credentials or customer data in the
report.

Expect a first response within a week. This is a personal project, not a staffed product — that is
the honest SLA.
