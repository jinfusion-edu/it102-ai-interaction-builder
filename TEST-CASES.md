# Test cases — 6 total (3 normal, 3 edge)

Required by the assignment: *"Test Cases (6 Total): You must present and walk
through a checklist of how your application handles different scenarios."*

The assignment supplies no table for this one, so these were authored.

> All six were **executed** by `node ../../tools/run_tests.js`, which loads the
> real `<script>` block from `index.html`, fires the actual click handler, and
> reads the actual `#output` paragraph. Results below are the harness's output,
> not predictions. Visual/layout behaviour was **not** tested — see the bottom
> of this file.

## Normal cases

### N1 — Both fields filled
- **Input:** Name `Alex`, Time `morning`
- **Expected:** `Good morning, Alex! Welcome back to JavaScript class.`
- **Actual:** ✅ **PASS** — exact match.

### N2 — A different time of day
- **Input:** Name `Sam`, Time `evening`
- **Expected:** `Good evening, Sam! Welcome back to JavaScript class.`
- **Actual:** ✅ **PASS** — exact match.

### N3 — Name given, time left blank (the specified default)
- **Input:** Name `Bo`, Time *(empty)*
- **Expected:** `Good day, Bo! Welcome back to JavaScript class.`
- **Actual:** ✅ **PASS** — the `"day"` default fired.
- **Why it is a normal case, not an edge case:** the assignment names this
  behaviour explicitly in the prompt, so it is specified functionality.

## Edge cases

### E1 — Both fields left blank
- **Input:** Name *(empty)*, Time *(empty)*
- **Expected:** `Good day, ! Welcome back to JavaScript class.` — the time
  default still fires; the name is genuinely absent.
- **Actual:** ✅ **PASS** — output exactly as above.
- **Honest note:** this output is grammatically broken. It is the *specified*
  behaviour (the prompt defines a default only for `time`) and is flagged as a
  known rough edge in `AUDIT-NOTES.md` rather than silently patched.

### E2 — Time field contains only whitespace
- **Input:** Name `Kim`, Time `"   "` (three spaces)
- **Expected:** `Good day, Kim! Welcome back to JavaScript class.` — whitespace
  is not a time of day.
- **Actual:** ✅ **PASS** — `.trim()` check caught it.
- **Why it matters:** this is the case a default parameter (`time = "day"`)
  cannot catch, and neither can a bare `if (time === "")`. It is the sharpest
  version of the assignment's "handle missing data" requirement.

### E3 — Very long name (150 characters)
- **Input:** Name = `x` × 150, Time `noon`
- **Expected:** no exception; a 196-character greeting
  (`"Good noon, "` = 11 + 150 + `"! Welcome back to JavaScript class."` = 35).
- **Actual:** ✅ **PASS** — length 196, no error thrown.
- **⚠ Limitation:** this proves the *string* is built correctly. It does **not**
  prove the page still looks right. The starter CSS caps the card at 400px with
  no `overflow-wrap`, so the text will very likely overflow horizontally.
  **Show this on camera** — it is the most likely visible defect in the project.

## Additional executed assertion

| Check | Result |
|---|---|
| `generateCustomGreeting("Ada", undefined)` → `"Good day, Ada! …"` | ✅ PASS |
| `getElementById` used for all four elements (4 occurrences) | ✅ PASS |
| `querySelector` not used (0 occurrences) | ✅ PASS |

## Explicitly NOT tested

- Any visual rendering — no browser was opened
- Whether a long name overflows the card (**expected to fail visually**)
- Keyboard activation of the button
- Touch devices
- Multiple rapid clicks (each click fully overwrites the output, so no state
  can accumulate — reasoned, not observed)
