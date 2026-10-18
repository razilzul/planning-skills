# planning-skills

A Claude Code plugin with five skills that all follow one idea: work out what to build, and agree on it, before any code changes.

| Skill | What it does |
|---|---|
| `start` | Plans a GitHub issue or an idea as one self-contained HTML page: mocks, clickable prototypes and diagrams, with every proposed change on a card you mark Want / Maybe / No. You paste your picks back and only the Want items get built. `--quick` gives the same keyed list in chat for small non-UI changes. |
| `redesign` | Audits an existing app screen or section and produces two HTML files: a report (real screenshots, severity-ranked findings, before/after structure, mockups in the project's own design tokens, feature ideas, a value-vs-effort matrix, a phased rollout) and an interactive phone demo with a "Your picks" export. |
| `simple` | Re-explains the last answer, a PR, an issue, a file or an error in plain words: what it is, what to pick, and before vs after, shown as a one-screen visual instead of paragraphs. |
| `tidy` | Cleans up a messy git checkout without losing anything. Classifies every uncommitted change, untracked file, stash and unpushed commit as discard / keep on a branch / gitignore / move / ask, asks once, backs everything up to a stash, then applies. |
| `grill-me` | Interviews you in plain text, round by round, in dependency order, until every consequential decision is settled and you confirm a shared-understanding summary. No implementation until you confirm. |

## Install

The repository is a plugin marketplace. In your shell:

```bash
claude plugin marketplace add razilzul/planning-skills
claude plugin install planning-skills@planning-skills
```

Or from inside a Claude Code session (v2.1.275 or later):

```
/plugin install planning-skills --marketplace razilzul/planning-skills
```

To update later: `claude plugin update planning-skills@planning-skills`.

## Use

Each skill triggers from its description when you ask for that kind of help ("plan issue 40 before building", "redesign the settings screen", "explain simply", "clean up this checkout", "grill me on this idea"). You can also call one directly, namespaced under the plugin:

```
/planning-skills:start 42
/planning-skills:redesign settings and onboarding
/planning-skills:simple
/planning-skills:tidy
/planning-skills:grill-me
```

`simple` is not the built-in `/simplify`, which cleans up code.

## What to expect

- **The repo's own rules win.** `start` and `redesign` read the project's `AGENTS.md` / `CLAUDE.md` first and follow its branch naming, ship channel and review rules over their own defaults.
- **HTML goes outside the repo by default.** `start` writes to `~/plans/<repo>/<issue-or-slug>/`, `simple` to `~/.simple/`, so your git status stays clean. `redesign` uses the repo's mocks folder if it has one, otherwise `design/redesign-<slug>/`.
- **Open the HTML in a real browser.** The pick pages autosave to `localStorage` and copy to the clipboard. Many in-app preview panes block both and reload on file change, which can wipe your marks. Every page also offers a plain-text export.
- **Verification uses headless Chrome.** `start` and `redesign` check their pages over the Chrome DevTools Protocol with Node's built-in `WebSocket`, so Google Chrome and Node 22 or later should be installed.
- **No code before your picks.** `start`, `redesign` and `grill-me` are read-only until you paste picks back or confirm.

## Layout

```
.claude-plugin/
  marketplace.json      marketplace catalog (one plugin, source "./")
  plugin.json           plugin manifest
skills/
  start/                SKILL.md, assets/plan-kit.html (pick cards, autosave, Markdown export)
  redesign/             SKILL.md, reference/example-demo.html (small neutral demo)
  simple/               SKILL.md
  tidy/                 SKILL.md
  grill-me/             SKILL.md, agents/openai.yaml (display metadata for other agent harnesses; Claude Code ignores it)
```

The `SKILL.md` files follow the open Agent Skills format, so the folders also work if you copy them into `~/.claude/skills/` or another harness that reads skills.

## License

MIT. See [LICENSE](LICENSE).
