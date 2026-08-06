# The AI-Powered Interaction Builder

IT102 · Introduction to Programming · Seattle Colleges

A single page that takes a name and a time of day and writes a greeting into
the document when a button is clicked.

## What it does

Two text inputs, a button, and an output paragraph. Clicking the button reads
both inputs, passes them to `generateCustomGreeting(name, time)`, and writes the
returned string into the page.

If the time field is left empty the function substitutes `"day"`, so the output
is always a complete sentence.

## Files

| File | Role |
|---|---|
| `index.html` | Everything — markup, the professor's inline `<style>`, and the `<script>` block |

The assignment supplies the starter file and instructs that the JavaScript go
*inside the `<script>` tags at the bottom of the HTML file*, so there is no
separate `.js` file by design. The starter markup and CSS are reproduced
unchanged.

## How to run it

```bash
git clone https://github.com/jinfusion-edu/it102-ai-interaction-builder.git
cd it102-ai-interaction-builder
```

Open `index.html` in a browser. No dependencies, no build step, no server
needed (though `python -m http.server 8000` works if you prefer).

## Expected output

A white card headed **Interactive Greeter** with two labelled fields and a blue
**Generate Greeting** button.

| Name | Time of day | Output |
|---|---|---|
| `Alex` | `morning` | `Good morning, Alex! Welcome back to JavaScript class.` |
| `Sam` | `evening` | `Good evening, Sam! Welcome back to JavaScript class.` |
| `Bo` | *(blank)* | `Good day, Bo! Welcome back to JavaScript class.` |

## Live URL

https://jinfusion-edu.github.io/it102-ai-interaction-builder/

## AI collaboration — tool and prompt

**Tool used:** Claude (Anthropic), via Claude Code.

The assignment specifies the exact prompt and instructs that the AI write the
function **only** — not the event listeners, because wiring those up is the
skill being practised.

> Act as a JavaScript coding assistant. Write a single JavaScript function named
> "generateCustomGreeting".
> It should accept two parameters: "name" and "time".
>
> Inside the function, it should return a string that looks like this:
> "Good [time], [name]! Welcome back to JavaScript class."
>
> Ensure the function handles missing data: if no time is provided, it should
> default to "day".
> Return only the JavaScript function code, no markdown explanations.

The event listener below the function was written by hand, per Step 3.

### What I corrected after reviewing the output

The important one: a default parameter (`function generateCustomGreeting(name, time = "day")`)
satisfies the prompt as written but **fails in this page**, because an empty text
input supplies `""`, not `undefined`, and defaults only fire for `undefined`.
The shipped version tests the value itself. Full reasoning in `AUDIT-NOTES.md`.

## Verification

```bash
node ../../tools/run_tests.js
```

Loads the real `<script>` block from this file, drives the actual click handler,
and reads the actual output paragraph. 7 assertions for this assignment, all
passing. See `TEST-CASES.md`.
