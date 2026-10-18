---
name: simple
description: Re-explain something in plain words for an owner who doesn't want to read. Turns the last answer, a plan, a PR, an issue, a file, an error or a doc into three things - what it is (one line), what I recommend (one line), and before vs after - shown as a visual (mock, diagram or side-by-side), not paragraphs. Works in any repo or folder. Use when the user invokes /simple, or says "too many words", "I don't understand", "explain simply", "ELI5", "just give me the summary", "TL;DR", "what does this mean", "make it easy for me". Not for code cleanup (that is the built-in /simplify).
argument-hint: "[what to explain: blank = your last answer, or a PR #, issue #, file path, link, or pasted text]"
---

# /simple: make it easy, show don't tell

The owner doesn't want to read. They want to know **what this is, what I should pick, and what changes**, and they understand pictures faster than words.

Target = `$ARGUMENTS`. If blank, use the last substantial thing in this conversation (usually my previous answer). If it's a PR / issue / file / link, read it first. Don't ask what to explain unless there is truly nothing to go on.

## Output: always these three, in this order

### 1. The visual (do this first, it's the main answer)

One self-contained HTML page that fits on one screen without scrolling where possible. Pick the form that fits:

| The thing is about... | Show |
|---|---|
| A screen, app or UI change | Before / After phone or desktop frames side by side, using the project's real theme colours, fonts and assets if I'm in a repo |
| A process, flow, code or system change | Before / After boxes-and-arrows diagram (what happens today vs after) |
| Choosing between options | 2-3 cards side by side, my pick highlighted with a "Recommended" tag, one line of trade-off each |
| Numbers, stats, costs, time | A simple bar or a big-number tile, before vs after |
| An error or problem | "What broke" -> "Why" -> "Fix" as three boxes |

Rules for the page:
- Top: one big line "What this is". Then the visual. Then a "My pick" box. Words on the page are labels, not paragraphs. Max ~12 words per label.
- Clear **BEFORE** and **AFTER** headings (or "Today" / "After"). Mark what changed with colour or a highlight, not with text.
- No emoji, no jargon. If a technical word is unavoidable, add a 3-5 word plain meaning in grey next to it.
- Light theme, big text, generous spacing. Use the repo's real tokens when there is a repo; otherwise neutral grey + one accent colour.
- Save it **outside any repo**: `~/.simple/<short-name>.html` (macOS) / `%USERPROFILE%\.simple\<short-name>.html` (Windows). Never write it into the project (keeps the project's git status clean).
- Show it: if your harness has an HTML preview or render tool, use it. Otherwise open it in Chrome: macOS `open -a "Google Chrome" <file>`, Windows `start chrome "<file>"`.

If the HTML is truly overkill (a yes/no thing), skip it and use a tiny markdown before/after table instead.

### 2. The chat reply: 5 lines max

```
**What it is:** <one plain sentence, like you'd tell a friend>
**My pick:** <one sentence: what to do and the one reason why>
**Before:** <one line>
**After:** <one line>
**Need from you:** <one yes/no or A/B question, or "nothing">
```

Don't repeat what the visual already shows. No "here's the page" announcements.

### 3. Close

End with a one-line status: `Status: <stage> · Next: <one line>`. If the turn also did real work (code, deploy, files) and the user's CLAUDE.md defines an end-of-turn handoff format, use it, but keep every cell to a few plain words.

## Writing rules (apply everywhere in this skill)

- Short words, short sentences. Write for a smart person who is tired and outside this topic.
- Replace jargon, don't define it: "merge" -> "add to the main code", "deploy" -> "put live", "API" -> "the connection to <thing>", "refactor" -> "tidy the code, no visible change".
- Numbers beat adjectives ("2 taps instead of 5", not "much faster").
- One recommendation, not a survey. Mention alternatives only on the visual as greyed-out cards.
- If I'm not sure of something, say "not sure" in plain words rather than hedging in long sentences.
- Never drop a real risk to keep it short. Put it as one red line on the visual: "Risk: <plain words>".

## Stickiness

After a /simple, keep answering in this style for the rest of the conversation unless the owner asks for detail ("more detail", "full version", "explain properly").
