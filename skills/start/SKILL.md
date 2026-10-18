---
name: start
description: Start work on a GitHub issue or an idea by planning first. Reads the issue, the repo rules and the code, then builds one self-contained HTML plan with high-fidelity mocks, clickable prototypes and diagrams, where the owner marks every proposed change Want / Maybe / No, adds notes, and pastes the picks back. Code starts only after the picks come back, and only for Want items. Use when the user invokes /start with an issue number or idea, or says "start issue #12", "plan issue 40", "let's start on …", "plan this before building".
argument-hint: "<issue # or short description> [--quick]"
---

# /start: plan first, as an HTML the owner marks up

Deliverable: **one self-contained HTML plan** the owner can read, click through and mark up. Every proposed change, option, extra and open question is a **pick card** with Want / Maybe / No and notes. The owner's pasted export is the implementation brief.

Until the picks come back: **no app code, no commits, no PR, no deploys.** Read-only research is fine, and so is moving the issue to "In Progress" if the repo's rules say starting work does that.

Target = `$ARGUMENTS`. An issue number (`#12` or `12`), or a short description of an idea. If it's empty, ask once in plain text.

## 1. Load the rules and the issue

1. Read the repo's `AGENTS.md` and `CLAUDE.md` (and anything they route to for this kind of work: runbooks, specs, design docs). Write down the **hard constraints** (e.g. no backend, zero runtime AI cost, owner-gated prod, content QA labels, OTA vs store build) and the repo's **issue workflow** (board status moves, branch naming, labels such as `release:*`).
2. **The repo's rules win over this skill.** If the repo is silent, the default is a branch named `<issue>-<slug>`, and never commit straight to the default branch.
3. For an issue: `gh issue view <n> --comments` plus linked PRs. For an idea: restate it in one sentence.

## 2. Plan with the built-in planning tools

- Use your normal planning mode (plan mode in Claude Code, or read-only exploration in other harnesses). Trace the real code paths, entry points, data model, existing utilities to reuse, tests, and how the change gets verified and shipped.
- Separate facts from decisions. Research facts yourself and never ask the owner for something you can look up. If one decision blocks the whole plan (the goal itself is ambiguous), ask up to 3 plain-text questions first. Everything else becomes an **Open question** pick card in the plan, each with your recommended answer.
- If there are genuinely different approaches, offer them as options A / B (/ C) with one recommended. Don't invent cosmetic alternatives.

## 3. Break the work into pick items

Give every item a stable key:

| Key | Kind | What it is |
|---|---|---|
| `O1…` | Option | An alternative approach. The owner Wants one of them. |
| `C1…` | Core change | Needed to meet the issue's goal. |
| `F1…` | Extra | Optional improvement worth doing alongside, checked against the hard constraints. |
| `Q1…` | Question | A decision the owner owns, with your recommendation pre-stated. |

Each item states: **What** (one line), **Why** (evidence: issue text, code, data, a guideline), **How it looks** (mock or prototype), **How it works** (flow or diagram), and tags: effort `S/M/L`, ship channel in the repo's terms (e.g. `OTA · JS-only`, `store build`, `backend migration`), risk, and the files it touches.

## 4. Build the plan HTML

Copy `assets/plan-kit.html` from this skill's folder. It already has the layout, the pick controls, autosave, Markdown export and copy buttons. Fill the sections and restyle the CSS variables to the product's brand (fonts, colours, radius, real assets).

**Where:** `~/plans/<repo>/<issue-or-slug>/index.html` (screenshots in `shots/` next to it), outside the repo so git stays clean. Use a repo mocks folder only if the repo's rules ask for it.

Sections (sticky table of contents; keep this order):
1. **Summary:** goal, in scope / out of scope, done-when, and KPI tiles (items to decide, core changes, extras, open questions). Make the counts match the cards.
2. **Today:** real screenshots of the current state for UI work (launch the app the way the repo does it, seed realistic state), or a code/flow map for non-UI work.
3. **Approach:** `O` cards side by side, with the recommendation marked.
4. **Changes:** one `C` card each, with its **high-fidelity mock or prototype** next to the what/why.
   - UI: phone or desktop frames built from the project's real tokens and assets. Where the flow matters, make it a **clickable prototype** (vanilla JS state + `render()`, real taps, sheets and toggles; anything out of scope is a labelled stand-in).
   - Non-UI: inline-SVG sequence or flow diagrams, before/after data-model tables, example payloads or API contracts, and a migration preview with row counts where it matters.
5. **Extras:** `F` cards with value and effort meters and a live preview where it's demoable.
6. **How it ships:** ordered implementation steps, files, tests and verification per the repo rules, release channel, and (if prod is touched) the repo's impact brief: old-version users, new-version users, risk, rollback.
7. **Open questions:** `Q` cards.
8. **Your picks:** the kit's summary, general notes, export and copy.

Style: no emoji anywhere (use real art or small inline SVG icons), short plain copy, invented data labelled "illustrative". Responsive, and it must still work at 980px.

## 5. Verify before handing off

Drive the page with headless Chrome over CDP (Node's built-in `WebSocket`). `<chrome>` is `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"` on macOS and `"C:\Program Files\Google\Chrome\Application\chrome.exe"` on Windows.
- Require zero JS exceptions and zero broken images (`[...document.images].filter(i=>!i.naturalWidth)`).
- Click through every prototype flow. Mark one pick, type a note and check that the export text shows it, then clear that test state (fresh profile or remove the key).
- Screenshot each section (set `scrollBehavior='auto'` before `scrollIntoView()`) and **look at the images**. Fix clipping, overlap and wrong facts.

## 6. Hand off, then stop

- Open the plan **in Chrome, not an in-app or harness preview pane**: macOS `open -a "Google Chrome" <file>`, Windows `start chrome "<file>"`. The preview blocks localStorage and reloads on edit.
- Reply with the clickable path, one line on how to use it (mark each card, add notes, **Copy my picks**, paste back here), your recommended picks, and anything not verified. Follow any end-of-turn format the user's CLAUDE.md requires.
- **Never edit the plan while the owner may be marking it.** If a fix is needed, ask first or have them copy their picks out.

`--quick` (for a change of roughly under 30 lines with no UI): skip the HTML. Post the same keyed items as a plain-text list in chat with a recommendation for each, and ask for picks.

## 7. After the owner pastes picks

The pasted export is the source of truth.
- Build exactly the **Want** items, in the chosen option. **Maybe** means not yet: list them and ask, or park them as a follow-up issue. Skip **No** items.
- Notes win over the plan. Ask only if a note is genuinely ambiguous.
- Now follow the repo's workflow: create the branch, move the board, implement in the plan's order, verify the way the repo requires (with screenshots for UI), open the PR, and link the PR to the issue.
- Report per key what shipped, what was skipped, and what's left.
