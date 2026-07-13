
## Linear PMO workflow (Trakko AI workspace, team TRA)

This repo feeds the Linear project **Studio Ops** via the native GitHub integration.
Every Claude Code session that changes code here MUST:

1. Find or create the Linear issue first (Linear MCP: list_issues / save_issue; project "Studio Ops", verb-first title <=70 chars, evidence-cited description).
2. Work on the Linear-generated branch name (the issue's gitBranchName, e.g. `lmc/tra-NN-slug`) — pushing it auto-moves the issue to In Progress.
3. Reference the issue in commits ("Part of TRA-NN") and PRs ("Fixes TRA-NN") — merging auto-closes it (~4 s latency, verified).

Secret hygiene: never commit or paste keys, tokens, or .env content into code, commits, or Linear. Full PMO conventions (sync-ref markers, issue budget): `~/trakko-pmo/CLAUDE.md`.
