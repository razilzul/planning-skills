---
name: grill-me
description: Relentlessly conduct a dependency-aware, plain-text decision-tree interview before planning or implementation. Use when the user asks to "grill me", "grill this idea", "ask me questions before implementing", "clarify the design", "work through the requirements", "reach shared understanding", or invokes /grill-me.
---

# Grill Me

Turn an underspecified request into a confirmed design by resolving every consequential user decision in dependency order. Optimize for reaching the right answer. Do not implement, edit, deploy, or otherwise act on the design until the user confirms the final shared understanding.

## Build the decision tree

1. Extract decisions already settled by the request and prior answers.
2. Represent every remaining consequential decision as a node with its prerequisite decisions.
3. Treat the frontier as all unsettled nodes whose prerequisites are settled.
4. Keep facts separate from decisions. Resolve facts from the filesystem, documentation, tools, or other available evidence; never ask the user to find information the agent can obtain.
5. Research directly unless delegation is explicitly requested or required by higher-priority instructions. Ask independent frontier questions while research is pending and defer only questions downstream of missing facts.

Recompute the tree after every completed round. An answer may add, remove, or reshape downstream branches.

## Ask in plain text

Always ask directly in the conversation. Do not call structured-question tools, form or elicitation tools, `AskUserQuestion`, HTML, visualizations, or external questionnaire pages.

Freeze the current frontier and ask every question in it in one plain-text round. Questions are open-ended, so every response is inherently a custom answer. Give one evidence-based recommendation for each question; do not present a closed multiple-choice menu.

Format each question exactly like this:

```text
Q1. <short title>: <open-ended question>

Recommended: <recommended answer>. <concise reason>
```

Separate questions with `---`. Tell the user they may reply `recommended` for any question or provide their own answer. Never silently accept a recommendation.

## Work in rounds

1. Freeze the current frontier before asking the round.
2. Ask every question in that frozen frontier, with stable numbering.
3. Wait until every question in the round is answered. If an answer is missing or ambiguous, ask only for that answer before advancing.
4. Apply the answers, research newly relevant facts, recompute the tree, and ask the next frontier.

Do not ask a question whose answer depends on another unsettled question in the same round. Put it in a later round.

After a round is answered, continue directly to the next nonempty frontier. Do not ask whether to continue or wait for a separate "go ahead." Pause only if the user asks to stop.

## Protect answer quality

- Recommend from evidence, constraints, reversibility, and downstream consequences, not personal preference.
- Surface contradictions between an answer and an already settled requirement.
- Explore materially different alternatives; do not create cosmetic choices to appear thorough.
- Test the emerging design against failure modes, edge cases, operational constraints, and acceptance criteria before declaring the frontier empty.
- Keep an explicit assumption list and either verify each factual assumption or turn each consequential preference into a decision.

## Reach confirmation

Continue until the frontier is empty and no consequential branch remains silently assumed. Then:

1. Present a compact shared-understanding summary covering the goal, scope, behavior, constraints, edge cases, assumptions, and acceptance criteria.
2. Ask one final open-ended plain-text confirmation question: `Does this shared understanding match your intent?`
3. Recommend `Confirm` when the summary is accurate, while accepting corrections or any custom response.
4. Begin planning or implementation only after explicit confirmation. If revisions are requested, update the tree and resume the rounds.

Do not confuse an empty frontier with confirmation; confirmation is a separate, required user decision.
