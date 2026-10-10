---
name: kickoff
description: Start your project here. Sets up any repo (empty or existing) for the werf and ends with an ordered backlog it can build. Use when the user types /werf:kickoff, or asks to start a new project or set up the werf in a repo.
---

# Kickoff: start your project here

You walk the user through the checklist below, in order. Every step starts
with its skip check: if the check passes, say `<step>: already done` in one
line and move on. So this runs on an empty folder, an existing app, or a repo
that is half set up.

Never overwrite a file that exists. Merge into it, or ask.

## 1. Preflight

Skip when: the folder is a git repo, `git branch --show-current` prints
`main`, and `git status --porcelain` prints nothing.

- Not a repo: `git init -b main`.
- On another branch: werf only works from `main`. If `main` exists, ask to
  switch to it. If not (e.g. the default branch is `master`), ask to rename it:
  `git branch -m <current> main`.
- Uncommitted changes: list them and ask whether to commit them now. Do not go
  on with a dirty tree; every werf job branches from `main`'s last commit.

## 2. Brainstorm the product

Skip when: `AGENTS.md` (or `CLAUDE.md`) already has a `## Product` section
and the user says it is still right.

Read the repo first (README, existing pages, package files) so you don't ask
what the code already answers. Then ask one question per message, multiple
choice where you can, recommended option first:

1. What is it, and who is it for? What should a visitor or user walk away
   with?
2. What must v1 do? Push for the smallest version that is worth shipping.
3. Look and tone: a few words, reference sites or products, light/dark,
   anything to avoid.
4. Stack, hosting and constraints. For an existing app, confirm what's there.
   For an empty folder, recommend one stack.
5. What is explicitly out of scope for v1?

Then write back a short summary in two parts, what the user said and what you
assumed, and let them correct it. Nothing is saved until they agree.

## 3. Scaffold (empty repos only)

Skip when: the repo already has an app that runs.

Create the smallest runnable app on the agreed stack yourself (not via the
werf): the official starter, one page, no extras. Install dependencies, then
run the dev server and the build and confirm both work. Commit it on `main`.

## 4. Project files

Skip each item that is already in place.

- `werf/backlog.md` with exactly:
  ```markdown
  # Backlog

  `/werf:run next` builds the first unchecked item and checks it off with its job id.
  ```
- `werf/dashboard.html`: copy `${CLAUDE_SKILL_DIR}/dashboard.html`.
- `werf/jobs/` exists (`mkdir -p werf/jobs`), so the dashboard has a folder to
  read before the first job.
- `.gitignore` contains the line `/werf/jobs/`.
- `.claude/settings.json` contains the block below, so anyone who clones and
  trusts the repo gets the werf. Merge it into existing JSON; keep every key
  that is already there.
  ```json
  {
    "extraKnownMarketplaces": {
      "werf": { "source": { "source": "github", "repo": "Tristandemuijnck/werf" } }
    },
    "enabledPlugins": { "werf@werf": true }
  }
  ```

## 5. AGENTS.md

The werf's agents carry nothing repo-specific; they read everything from
`AGENTS.md` (or `CLAUDE.md` if that is what the repo uses). It must give:

- **Setup** for a fresh checkout or worktree (e.g. `npm ci`).
- **Dev server** on a given port (e.g. `npm run dev -- --port <port>`).
- **Validation**: tests, build, lint, type-check.
- **`## Product`**: the brainstorm summary from step 2 (what, who, v1, look
  and tone, stack, out of scope).

Only write commands you ran in this session and saw succeed. Run the dev server
on a spare port, load the page, stop it. If tests or a linter don't exist, say
so in one line; don't invent them. If the file exists, fill only what's
missing.

## 6. Prerequisites

Skip when: `agent-browser --version` works.

Otherwise run `npm install -g agent-browser`, then `agent-browser install`
(downloads Chrome). The builder and the UX/UI reviewers use it to look at
pages.

## 7. Backlog

Skip when: `werf/backlog.md` already has `- [ ]` items and the user wants to
keep them.

Turn v1 into one-line items, one feature each. Rules:

- Imperative and concrete: `Add a projects page listing work from content files`.
- One item is one thing a reviewer can judge. Split anything that needs
  "and then".
- Foundations first: layout, navigation, design tokens, data model. Jobs
  branch from `main` when they start, so a later item can only build on
  earlier items that have merged.
- Leave out anything the user put out of scope.

Show the list and let the user reorder, cut or add. Then append it to
`werf/backlog.md` as `- [ ] <item>` lines.

## 8. Commit and hand off

Commit everything from this run on `main` in one commit:
`Kick off werf: AGENTS.md, backlog, dashboard`.

Then tell the user:

- Next: `/werf:run next` builds the first backlog item. Open one session per
  job to run several in parallel.
- Dashboard: `cd werf && python3 -m http.server 8000` (`python` on Windows),
  then open http://localhost:8000/dashboard.html.
