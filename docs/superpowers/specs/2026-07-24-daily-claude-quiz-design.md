# Daily Claude Feature Quiz — Design

## Purpose

Add a small daily learning card to the existing single-page todo app (`todo.html`) so the user is regularly exposed to real Claude/Anthropic features, functionalities, and capabilities, and can apply them in daily work.

## Placement

A new card is inserted at the top of the "To Do" (Today) pane (`#pane-todo`), above the filter bar. It renders only when the currently viewed date (`selDate`) equals today (`todayStr()`) — it does not show when browsing past or future days via the date nav, since it is inherently a "today" feature.

## Content model

A hardcoded JS array, `CLAUDE_QUIZ`, added to the `<script>` block. Starting size: ~30-35 entries covering real Claude/Anthropic capabilities (Claude Code, Agent SDK, computer use, Artifacts, Projects, extended thinking, MCP, vision/PDF support, prompt caching, tool use, Claude in Slack, etc). Each entry:

```js
{
  en: { fact: "...", question: "...", choices: ["A","B","C"], answerIndex: 1, explanation: "..." },
  am: { fact: "...", question: "...", choices: ["...","...","..."], answerIndex: 1, explanation: "..." }
}
```

- `fact`: 1-2 sentence blurb introducing the feature.
- `question` / `choices` / `answerIndex`: a multiple-choice question (3-4 options) about the fact.
- `explanation`: shown after answering, regardless of correctness.

The list is intended to be appended to over time as new Claude features ship; no mechanism is needed to fetch content remotely (page is static/client-side).

## Daily selection

Deterministic, no state needed to pick the entry:

```js
function dayOfYear(dateStr) { /* 1-based day number within selDate's year */ }
const quizIndex = dayOfYear(todayStr()) % CLAUDE_QUIZ.length;
```

Same question shows all day, changes at local midnight, cycles through the full list before repeating.

## UI / interaction

Card structure (new HTML block, styled consistent with existing `.stats-bar`/card patterns):

- Header: "🧠 Claude Feature of the Day" (localized via `STRINGS`).
- Fact blurb text.
- Question text.
- Choice buttons (one per option, full-width stacked or wrapped row, matching `.filter-btn`-like styling).
- On choosing an option:
  - Locks in the choice (buttons become non-interactive for this question).
  - Highlights the selected button green (correct) or red (incorrect) via class, and separately highlights the actual correct answer green if the user was wrong.
  - Reveals the `explanation` text below the choices.

## Persistence

`localStorage` key `dtodo_quiz_v1`, JSON map of `{ "YYYY-MM-DD": choiceIndex }`. On render:
- If today's date has a stored entry, render the card already in the "answered" state (correct/incorrect coloring + explanation visible), reconstructed from `CLAUDE_QUIZ[quizIndex]` and the stored `choiceIndex`.
- If not, render the unanswered state with clickable choices.

No Supabase table/sync — this is local-only, per-device state, consistent with being a lightweight learning nudge rather than task data.

## Language

Fully wired into the existing `lang` / `STRINGS` / `applyLang()` mechanism:
- New keys added to `STRINGS.en` and `STRINGS.am` for the card's static labels (title, "Correct!", "Not quite — here's why:", etc).
- Quiz entry content itself carries its own `en`/`am` fields (see Content model) and is re-rendered on language toggle just like the rest of the UI.

## Explicit non-goals

- No scoring, streak tracking, or history of past days' quizzes.
- No new tab/section — everything lives in one card on the existing Today view.
- No server-side/Supabase persistence of quiz answers.
- No automated content fetching — the list is manually curated/extended in code.

## Testing

Manual verification in a browser (this app has no automated test suite):
- Card appears on the Today pane when `selDate === todayStr()`, and is absent when navigating to another day.
- Answering a choice locks the question, shows correct/incorrect coloring and explanation.
- Reloading the page on the same day preserves the answered state.
- Toggling EN/ՀԱՅ re-renders the card's labels and quiz content in the matching language, including in the already-answered state.
- Content and layout look correct on both mobile-width and desktop-width viewports (app is responsive via existing breakpoints).
