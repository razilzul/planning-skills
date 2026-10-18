---
name: redesign
description: Audit an existing app screen or section (e.g. "settings", "parents area", "onboarding", "checkout") and produce two self-contained HTML files - a redesign proposal report and an interactive phone demo the owner can click through and mark up. The report includes real screenshots of the current state, severity-ranked findings, before/after information architecture, phone-frame mockups built from the project's own design tokens and assets, a "why" and an effort tag for every change, extra feature ideas checked against the project's constraints, a value-vs-effort matrix and a phased rollout. The demo renders each proposed direction live in a working phone frame with real interactions, preview switches for every optional feature, and a "Your picks" section (Want / Maybe / No + notes per change) that autosaves and exports Markdown picks the owner pastes back as the implementation brief. Use when the user invokes /redesign, or asks to "redesign", "revamp", "improve the look/UX of", "give me mock ideas for", or "audit the UI of" a screen or section.
argument-hint: "<screen or section to redesign, e.g. 'settings and parents section'>"
---

# /redesign: screen/section redesign proposal

Deliverable: **two self-contained HTML files**, both required:

1. **The report** (`<slug>-redesign.html`): what the target looks like today, what's wrong with it, mockups of the fix, why each change helps, and what else is worth building.
2. **The interactive demo** (`<slug>-demo.html`): a working phone the owner can actually try, one per proposed direction, plus a "Your picks" section where they mark each change Want / Maybe / No, leave notes, and copy a Markdown export back into chat. Static mocks alone are not enough: the owner needs to see how the changes actually look and feel.

It is a proposal only. **Do not modify app code, commit or open a PR** until the owner has pasted their picks back (step 9) or asks separately.

**Where files go:** if the repo already has a mocks folder (e.g. `docs/mocks/`), use it: `docs/mocks/<slug>-redesign.html`, `docs/mocks/<slug>-demo.html`, `docs/mocks/shots/<slug>-redesign/`. Otherwise default to `design/redesign-<slug>/` (`index.html`, `demo.html`, `shots/`). Wherever this skill says `design/redesign-<slug>/`, read it as the chosen folder.

**Owner style rules (report, demo and any UI copy):**
- **No emoji anywhere.** Use the project's real PNG art, or simple inline SVG icons (stroke icons built from a tiny `svg()` helper are fine).
- Short, plain copy. No filler pep lines ("Let's go!", "You've got this!") unless the product already uses that exact line.

Target = `$ARGUMENTS`. If empty, ask which screen or section, once, in plain text.

## 1. Ground yourself (read before judging)

1. Read the project's rules and product docs: `CLAUDE.md`, `AGENTS.md`, `SPEC.md`/PRD, brand/design docs (`docs/BRAND*`, `design/README*`). Write down the **hard constraints**, such as "local-first", "no accounts", "no runtime AI", pricing model, platforms, privacy rules and store-review rules. Every suggestion must respect them, and the report must say so.
2. Find the target code: grep for screen names, titles and nav routes. Read **every** screen in scope **and its entry points**. Trace how a user reaches it (router/`setScreen`/navigation stack) and where "back" goes. Buried or misrouted entry points are often the biggest finding.
3. Read the design system: the theme/tokens file (colors, gradients, fonts, radii, shadows), shared components (buttons, toggles, cards) and asset folders (mascots, icons, illustrations). The mockups **must** reuse these real tokens and assets through relative paths.
4. Pull real numbers from code/content instead of guessing (counts per level, economy constants, feature flags, existing links like privacy/terms, and whether restore-purchases exists). Mark any invented mock data as "illustrative".

## 2. Capture the current state (real screenshots)

- Launch the app the way the project does it. Use a project run skill if one exists, otherwise the dev server (e.g. `npx expo start --web --port <free port>`, `npm run dev`). Install deps in a worktree if `node_modules` is missing.
- Seed **realistic state** (a named user mid-way through, some history, a balance) via localStorage/AsyncStorage keys read from the storage module, so screens aren't empty. Seed PINs/flags needed to get past gates.
- Drive to each screen and save screenshots into `design/redesign-<slug>/shots/NN-name.png`. For long scrolling screens, use a tall viewport (e.g. 390×2900) or several scrolled shots.
- Tooling gotchas:
  - If your harness has a browser-preview snapshot tool, it can return huge accessibility trees. Prefer its image-off, save-to-file mode and copy the saved PNG path. If output overflows to a file, extract the screenshot path with `jq`/python.
  - For static HTML, headless Chrome is the most reliable full-page capture: `<chrome> --headless=new --hide-scrollbars --window-size=1280,9000 --virtual-time-budget=5000 --screenshot=out.png <url>`. `<chrome>` is `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"` on macOS and `"C:\Program Files\Google\Chrome\Application\chrome.exe"` on Windows (fallback `%LOCALAPPDATA%\Google\Chrome\Application\chrome.exe`).
  - `--screenshot` ignores `#anchors`. To capture one section, drive the page over CDP and set `document.documentElement.style.scrollBehavior='auto'` before `scrollIntoView()`, or smooth scrolling will animate past the capture.
  - For many interactive captures (tap through tabs, sheets, PIN pads), a tiny Chrome DevTools Protocol script is far cheaper than preview snapshots: launch Chrome with `--headless=new --remote-debugging-port=<port>`, connect with Node's built-in `WebSocket`, then `Runtime.evaluate` (seed localStorage, `element.click()` by aria-label or text) and `Page.captureScreenshot` with `Emulation.setDeviceMetricsOverride` (390×844 @2x; raise the height for full-scroll shots). Call `Network.clearBrowserCache` first. If a code edit doesn't show up, restart the bundler with `--clear` (Metro can miss edits made via `sed -i`).
  - Stop any dev servers you started when you're done.
- If the app truly can't run, fall back to existing screenshots in the repo (`design/screens/`) and **say so in the report**, including their date.

## 3. Audit with these lenses

Rate each finding **HIGH** (costs trust, money, safety or compliance), **MED** (friction) or **LOW** (polish). Aim for about 12–20 concrete findings, each with the evidence ("13 taps to give 140 coins", "toggles have empty accessible names").

- **Information architecture:** where things live, how many taps, gates in the wrong place, daily tasks mixed with reference, admin or dev material.
- **Functional gaps:** dead or read-only rows that look tappable, missing edit/undo, missing standard settings (profile edit, change PIN/password, restore purchases, privacy/terms/support, notifications, accessibility).
- **Data presentation:** discouraging denominators, raw ISO dates, unexplained empty states, lists that show results without insight.
- **Monetisation and trust:** paywall/upgrade cards with no call to action, legal links, dev/test copy that leaked into production ("handy for testing", third-party AI mentions).
- **Visual consistency:** more than one design language, emoji used where brand art exists, inconsistent language/bilingual rules, hierarchy, spacing, density.
- **Accessibility:** roles/labels (check the a11y tree), touch-target size, contrast, text scaling, motion.
- **Audience fit:** kid vs adult surfaces, novice vs power user, platform conventions and store guidelines.

## 4. Design the fix

- Start with an **information-architecture before/after**, since structure changes carry the most weight.
- Mock **4–7 screens** as HTML phone frames (≈300×640, status bar, header, body) using the project's real fonts, colors, gradients, radii and asset images. Check each mock for overflow (`scrollHeight > clientHeight`). Trim it or add a bottom fade so it reads as scrollable.
- Beside each mock, list numbered changes. Each one gets **What** (bold) and **Why** (user benefit, evidence, or a rule/guideline it satisfies), plus tags:
  - Priority phase (Phase 1/2/3)
  - Shipping cost in the project's terms (e.g. **OTA · JS-only** vs **needs store build / new native dep**, or **frontend only** vs **needs backend/migration**)
- Keep all copy in the product's language rules. Any new user-facing text in a non-English language is a **placeholder pending the project's content QA**. Say so, and never claim approval that didn't happen.

## 5. Suggest additional features

8–12 feature cards, each with: an icon (real asset PNG or inline SVG, never emoji), name, what it does, **Why** (the user or business outcome), and value and effort meters. Check each one against the hard constraints from step 1. For example, a "weekly report" in a no-backend app means share-sheet export, not email. Prefer features that reuse data or assets the app already has. Give each feature a stable `key`; the demo reuses the same list.

## 6. Build the HTML report

Path: see "Where files go" above, with `shots/` next to it. Single file, inline CSS, no JS frameworks. Google Fonts is fine if the brand font is a Google font. Sections, with a sticky table of contents:

1. **Hero**: title, one-paragraph summary, KPI tiles (screens audited, issues found, how many HIGH, mocks, feature ideas). Make the counts match the content.
2. **Today**: the real screenshots in scrollable frames, each with a caption.
3. **What gets in the way**: the severity-tagged findings grid.
4. **New structure**: the IA before → after.
5. **Mockups**: alternating phone and "why" columns.
6. **Features worth adding**: the cards.
7. **Where to start**: a value/effort 2×2 with every idea placed on it (quick wins highlighted).
8. **Rollout**: 3 phases, plus a "guardrails kept" note that restates the constraints.

Style: match the product's brand (palette, font, radius, mascot) so the report feels like part of the app. Keep it readable, lay it out as a responsive grid, and make sure it still works at 980px. Link to the demo file from the hero.

If you propose more than one visual direction, label them A / B / C in the report with one recommended; the demo uses the same labels.

## 7. Build the interactive demo (required)

Single self-contained HTML file, vanilla JS, no frameworks, no build step. Reference implementation: `reference/example-demo.html` in this skill's folder, a deliberately small demo for a made-up notes app's settings screen. It shows the script order, the `S` / `fresh()` / `render()` pattern, a PIN gate, preview switches and the picks export. Study it for code patterns; don't copy its content. A real demo is far richer and uses the project's own tokens and assets.

**Structure of the script** (keep it in this order so it stays editable):
- Data tables at the top: an asset root (`const A = '../../assets/'`, relative to the file), `DIRS` (per direction: title, one-line summary, `try` steps, `ch` "what changed" list), `CORE` (core changes) and `FEATS` (extra features: `key, name, img, what, why, tags, def, demo`), plus a table for any key policy decision (e.g. `PINROWS`: which settings need a PIN).
- One state object `S` from a `fresh()` factory (direction, current tab/screen, open sheet, toggles, seeded numbers, `feat` on/off map). `render()` rebuilds the phone's innerHTML from `S`; a separate `renderOverlay()` handles sheets and PIN pads. One delegated click handler on `[data-act]` inside the phone.
- Seed realistic data from the project's real content (real lesson/unit/item titles, real counts, economy constants). Use the same seeded user name and numbers as the report.

**Page layout:**
- **Top bar:** a direction switcher (A/B/C segmented control) that rebuilds the phone live; a **Compare with today** toggle that shows the real current screenshot (from `shots/`) next to the phone at the same scale; a **Reset demo** button (resets `S` but keeps direction and feature switches). Add any one-off replay button a direction needs (e.g. "Replay launch").
- **Phone:** ≈376×800 frame with status bar, built from the project's real tokens (fonts, colors, gradients, radii, shadows) and real asset images. Scale it to fit the viewport height (`scale = clamp(0.6, (innerHeight - 120) / 800, 1)`, recomputed on resize). Screens inside scroll like the real app.
- **Side panel (per direction):** title + summary; **Try this** numbered steps, each with a **Show me** button that drives the phone straight to that state (opens the sheet, scrolls the map, starts the lesson); a **What changed from today** list; and **live preview switches** for every optional feature so each can be seen on and off in the phone.

**Interactions:** every proposed flow must really work, not just be drawn. Taps open sheets, toggles and sliders change state, gates and PIN pads accept a PIN stated on screen (e.g. "demo PIN 1234") and reject wrong ones. Anything out of scope gets a simple stand-in screen clearly labelled **"stand-in, unchanged"**. Actions that would leave the app (store, links, purchases, share sheet) show a short toast describing what the real action would do ("Opens the privacy policy").

**"Your picks" section** (below the phone):
- **Direction pick** (A / B / C / Mix) + notes. Picking a direction also switches the phone to it.
- **Key policy decision** as an editable table pre-filled with the proposal (e.g. each settings row: PIN / Open + note). Mark rows the owner changed as "changed", and make the phone obey the table live.
- **Each core change and each extra feature** as a card: icon, name, what + why, phase/cost tags, **Want / Maybe / No** buttons (tap again to clear), a notes textarea, and a **Preview** switch where the feature is demoable.
- A **general notes** box.
- **Autosave** to `localStorage` under a versioned key (`<project>-<slug>-demo-picks-v1`), every read/write wrapped in `try/catch`. Show a save-status chip: green "Saved in this browser", or red "Not saving here: copy before you close" if storage fails.
- **Live Markdown export** in a visible readonly textarea, rebuilt on every change: direction + notes, the policy table (flag rows "CHANGED from proposal"), core changes and features each as `- Name: WANT/MAYBE/NO/(unmarked) · notes`, then other notes. A **Copy my picks** button tries `navigator.clipboard.writeText`, then `document.execCommand('copy')`, then selects the text and says "the text is selected, press ⌘C". Also a **Clear picks** button behind a confirm.
- Tell the owner in the section intro: the pasted export is what gets built.

## 8. Verify, then hand off

- **Report:** serve the folder (macOS `python3 -m http.server <port> --bind 127.0.0.1`, Windows `py -m http.server <port> --bind 127.0.0.1`). Check all images load (`[...document.images].filter(i=>!i.naturalWidth)`), check mock overflow, take a full-page screenshot and **look at it**. Fix clipping, wrapping and factual errors (re-check any product claim against the code, such as the free-tier rules or what's already built). Then stop the server.
- **Demo:** drive it with a headless-Chrome CDP script (same pattern as step 2). Collect `Runtime.exceptionThrown` and console errors and require **zero JS exceptions** and **zero broken images**. Click through every flow for every direction (each "Show me", each sheet, PIN right and wrong, each preview switch, a vote + note in the picks, the export text), and `Page.captureScreenshot` the phone clip at each step. **Open and look at the screenshots** before handing off; fix clipping, overlap and wrong copy. Check that the export reflects the clicks you made, then clear that test state (fresh profile or remove the key) so the owner starts clean.
- **Hand-off:** open the demo in Google Chrome: macOS `open -a "Google Chrome" "file://<absolute path>/<slug>-demo.html"`, Windows `start chrome "file:///<absolute path>/<slug>-demo.html"`. Do **not** use an in-app or harness preview pane: those block localStorage and clipboard, and auto-reload on file change (that once wiped an owner's marks). **Never edit the demo file while the owner may be marking it**; if a fix is needed, ask first or tell them to copy their picks out.
- Reply with: both file paths (clickable), how to use the demo (switch directions, "Show me", mark picks, Copy my picks, paste back here), 3–5 headline findings, the recommended first phase, and anything not verified (e.g. "couldn't run iOS, screenshots are web"). Follow any end-of-turn format the user's CLAUDE.md requires.
- Don't commit the report or demo unless asked. If asked, put them on a branch and open a PR per the repo's conventions.

## 9. After the owner pastes picks

The pasted export is the source of truth.
- Implement exactly the **Want** items, in the picked direction. Treat **Maybe** as not yet: list them and ask, or park them. Skip **No** items.
- Respect every note and rename in the picks (e.g. a label change, a PIN row flipped to "Open", "do this but smaller"). Where a note conflicts with the proposal, the note wins; ask only if it's genuinely ambiguous.
- Follow the repo's ship rules (branch/PR/merge, OTA vs store build, content QA labels) and the user's CLAUDE.md end-of-turn format. Verify on device/emulator the way the project requires.
