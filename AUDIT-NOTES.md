# Audit notes

Written to be attacked.

This assignment does not supply a named audit checklist (unlike the resume and
theme-toggle assignments). Its graded structure is Steps 1–3, so this file is
organised against those, then everything they do not cover.

---

## Against the assignment's steps

### ▸ Prerequisite — starter code reproduced exactly

**Yes, byte-for-byte.** The `<style>` block, `<div class="card">`, both labelled
inputs, `#greetBtn` and `<p id="output">` are unchanged from the assignment PDF.
Only the `// YOUR JS CODE WILL GO HERE` comment was replaced.

**Tension worth naming:** this starter uses `<div class="card">` and an inline
`<style>` block — both of which the *resume* assignment in this same course
explicitly forbids ("Do NOT use generic `<div>` … Do not use inline styles").
Nothing was "fixed", because rewriting professor-supplied scaffolding would be
deviating from the prerequisite. If a grader cross-applies the resume's rules
here, this looks like a violation. It is a deliberate choice to follow the
assignment in front of me.

### ▸ Step 1 — the function

- Named `generateCustomGreeting` ✅
- Two parameters, `name` and `time` ✅
- Returns `"Good [time], [name]! Welcome back to JavaScript class."` ✅ — verified
  by string equality in the test harness, not by eye
- Defaults to `"day"` when no time is provided ✅ — see the caveat below
- Contains no event listeners and no DOM access ✅

### ▸ Step 2 — pasted inside the `<script>` tags at the bottom of the HTML

**Yes.** No separate `.js` file, per the instruction.

### ▸ Step 3 — the event listener (my own work)

- `document.getElementById()` used for all four elements ✅ (grep: 4 occurrences,
  0 uses of `querySelector`)
- `'click'` listener on `#greetBtn` ✅
- `.value` of both inputs read **inside** the callback ✅
- `generateCustomGreeting()` called with those values as arguments ✅
- `.textContent` of `#output` updated with the return value ✅

---

## Beyond the steps

### Assumptions

1. The grader opens `index.html` directly, or serves it. Both work; nothing is
   fetched over the network.
2. JavaScript is enabled. Without it the card renders and the button is inert.
3. "No time is provided" includes an empty input box. This is an
   **interpretation**, argued below — it is the single most consequential
   judgement call in this repo.
4. The grader does not require the output to be cleared between clicks. Each
   click overwrites the paragraph entirely, so stale text cannot persist.

### The interpretation that matters

The prompt says *"if no time is provided, it should default to `day`"*. Read
strictly as a function contract, "not provided" means the argument is
`undefined`, and the idiomatic implementation is a default parameter:

```js
function generateCustomGreeting(name, time = "day") { ... }
```

**That implementation would fail this page.** `timeInput.value` on an empty text
box is `""`, not `undefined`, so the default never fires and the user sees
`"Good , Bo! Welcome back to JavaScript class."`.

The shipped code treats `undefined`, `null`, empty and whitespace-only as
"not provided". I believe this is what the requirement means, because an empty
field is how a user expresses "I didn't give you a time".

**The counter-argument, stated fairly:** a grader auto-checking
`generateCustomGreeting("Bo")` in a console would get `"Good day, Bo!..."` from
*both* implementations, so both pass that test. A grader who specifically wants
to see the ES6 default-parameter syntax would not see it here. If that is the
expectation, this is a miss — and I would still make the same call, because the
UI behaviour is the thing the user experiences.

### What I executed vs. what I only reasoned about

**Executed** — `node ../../tools/run_tests.js` extracts the real `<script>`
block, runs it against a DOM shim, fires the actual click handler and reads the
actual `#output` element. 7 assertions, all passing:

| Case | Input | Result |
|---|---|---|
| both fields filled | `Alex` / `morning` | PASS |
| second normal case | `Sam` / `evening` | PASS |
| blank time defaults | `Bo` / `""` | PASS — `"Good day, Bo! ..."` |
| both blank | `""` / `""` | PASS — `"Good day, ! ..."` |
| whitespace-only time | `Kim` / `"   "` | PASS — defaults to `day` |
| argument genuinely absent | `("Ada", undefined)` | PASS |
| 150-character name | length 196 output | PASS — no throw |

**Reasoned about, NOT executed:**
- **Any visual rendering.** No browser was opened. Card layout, the professor's
  CSS, and how a 150-character name actually behaves *on screen* are unverified.
  The long-name test proves the string is produced, **not** that the card holds
  its shape — see the unhandled edge cases below.
- **Keyboard activation** of the button.
- **Behaviour on a touch device.**

### Edge cases known to be unhandled

- **Blank name.** `"Good day, ! Welcome back to JavaScript class."` — grammatically
  broken. Deliberate: the prompt specifies a default only for `time`. One line
  would fix it (`if (!name.trim()) name = "friend";`) and was **not** added,
  because inventing behaviour the spec does not describe is its own kind of error.
- **Long input overflowing the card.** The starter CSS sets `max-width: 400px`
  with no `overflow-wrap`. A 150-character unbroken name will very likely
  overflow the card horizontally. The test proves no exception is thrown; it
  proves nothing about layout. **This is the most likely visible defect.**
- **Capitalisation is not normalised.** `Morning` → `"Good Morning, ..."`.
  Intentional, per the exact output template.
- **Nonsense time values** are accepted verbatim: `"banana"` →
  `"Good banana, Alex! ..."`. There is no allow-list, and the spec asks for none.
- **No output is shown before the first click.** The paragraph starts empty. The
  assignment does not ask for placeholder text.
- **`.value` is not trimmed for the name**, so `"  Alex  "` renders with padding
  inside the sentence.

### Three places I would look first if this turned out to be wrong

1. **The empty-string default branch.** If the greeting ever renders as
   `"Good , Bo!"`, the `.trim() === ""` test is the thing that failed or was
   edited away — most likely by someone "simplifying" it into a default
   parameter.
2. **Element id spelling.** `userName`, `timeOfDay`, `greetBtn`, `output` are
   magic strings duplicated between the HTML and the script. A typo yields
   `null` and the first click throws `Cannot read properties of null (reading
   'value')`. Any "nothing happens when I click" report starts here.
3. **Script position.** The `<script>` block is the last thing in `<body>`. If it
   were moved above the card, all four `getElementById` calls would return
   `null` at load.

### What I would flag reviewing this as someone else's code

- `var` throughout instead of `const`/`let` — deliberate for the course level,
  but a modern reviewer would flag it on sight.
- The greeting template is a bare string literal inside the function. If the
  wording ever needs to change it must be edited in one place, which is fine —
  but the test harness hardcodes the same sentence, so a change requires editing
  two files, and the test would catch it. Acceptable, worth knowing.
- No null-guard on any selector. Fine for a teaching exercise; unacceptable in
  production.
- `String(timeOfDay).trim()` is slightly defensive for a function that will only
  ever receive strings from this page. I kept it because the function is public
  and testable in isolation, but a reviewer could reasonably call it noise.

### Nothing found clean

Not clean. The long-name overflow is a real unverified risk, the blank-name
output is genuinely ungrammatical, and the empty-string-vs-`undefined`
interpretation is a defensible call that a different grader could mark
differently.
