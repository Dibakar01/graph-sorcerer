# Changelog

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning is [Semantic](https://semver.org/spec/v2.0.0.html).

## [0.1.0] — 2026-08-27

First release.

### Added
- `graph-sorcerer` skill: gates a job with the fake-edge test, designs the diamond, presents for
  approval, and runs only on an explicit go-ahead.
- `references/patterns.md`: verified Claude Code mechanics, the diamond skeleton, fixes for the
  three ways graphs break, and six paste-ready spec templates.
- Three eval cases for `claude plugin eval`, including `refuses-narrow-work` — the case the plugin
  is meant to *decline*.

### Notes
- Mechanics were verified against the CLI changelog rather than taken from the source article.
  Three of the article's instructions were out of date; the corrections are in the README.
- Always-on cost is ~130 tokens per session, measured with `claude plugin details`.
