# werf

A software shipyard for [Claude Code](https://claude.com/claude-code). *Werf* is
Dutch for shipyard: you hand it a one-line feature, and it comes back specced,
built, reviewed and merged.

```
feature → spec-writer → builder → [security, ux, ui, code] → WAIT FOR ALL
                           ▲                                      │
                           └──── any CHANGES: resume builder ◄────┤
                                                                  ▼ all PASS
                                     merge to main ◄─ APPROVE ─ approver ─ ESCALATE → needs-human
```

Every job runs in its own git worktree and branch (`werf/<job-id>`), so you can
run several at once, one Claude Code session each.

## Install

```
/plugin marketplace add Tristandemuijnck/werf
/plugin install werf@werf
```

No SSH key for GitHub? Installs from inside a session use SSH. Set
`CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1`, or install from your shell with
`claude plugin marketplace add Tristandemuijnck/werf && claude plugin install werf@werf`.

Needs git, Python 3 (for the dashboard) and, for the builder and UX/UI reviewers, the
[agent-browser](https://github.com/vercel-labs/agent-browser) CLI (`/werf:kickoff`
installs it if it's missing).

## Start a project

In any repo, empty or existing:

```
/werf:kickoff
```

It checks git, brainstorms the product with you, scaffolds an app if there is
none, writes `AGENTS.md` (setup, dev server, validation, product notes), adds
`werf/backlog.md` and `werf/dashboard.html`, enables the plugin for the repo in
`.claude/settings.json`, and fills the backlog.

## Run jobs

| Command                             | Does                                          |
| ----------------------------------- | --------------------------------------------- |
| `/werf:run <feature>`               | Builds one feature end to end                 |
| `/werf:run next`                    | Builds the first unchecked backlog item       |
| `/werf:run approve <job-id>`        | Merges a job waiting on a human               |
| `/werf:run rework <job-id> <note>`  | Sends a waiting job back to the builder       |

Jobs that pass every review merge into `main` locally. The werf never pushes.

## Dashboard

```
cd werf && python3 -m http.server 8000   # python on Windows
```

Open http://localhost:8000/dashboard.html to watch jobs move through the loop.

## Update

```
/plugin marketplace update werf
```

To change the werf itself, edit `skills/` and `agents/` in this repo.

## License

MIT
