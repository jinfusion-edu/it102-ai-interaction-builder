# How this works, and why

---

## 1. The concept being tested

**Wiring a pure function to a live interface.**

The assignment deliberately splits the work in two and gives the harder-looking
half to the AI:

- `generateCustomGreeting(name, time)` — takes values in, returns a string out.
  It never touches the page.
- The event listener — reads the page, calls the function, writes the result
  back.

That split is the lesson. The function is **pure**: same inputs, same output,
no side effects. You can test it without a browser (this repo does exactly
that). The listener is the *impure* part — it knows about the DOM, about
`#userName`, about when a click happened.

Keeping them separate means the logic can be reasoned about and tested in
isolation, and the messy world-facing code stays thin. The prompt enforces this
by explicitly telling the AI **not** to write the event listeners.

---

## 2. Walking the control flow

```
user clicks #greetBtn
        │
        ▼
callback runs
        │  nameInput.value   ──► "Alex"
        │  timeInput.value   ──► "morning"
        ▼
generateCustomGreeting("Alex", "morning")
        │  returns "Good morning, Alex! Welcome back to JavaScript class."
        ▼
outputParagraph.textContent = <that string>
        │
        ▼
browser repaints; the sentence appears
```

### Step 1 — selecting

```js
var nameInput = document.getElementById("userName");
```

`getElementById` finds the one element whose `id` matches. Ids are unique in a
document, so it returns a single element or `null`.

Selection happens **once**, at load, not inside the click handler. The elements
do not change, so there is no reason to look them up again on every click.

> Note: the previous assignment (2 Tone Theme Toggle) required `querySelector`
> and treated `getElementById` as an outdated method to correct. This one
> explicitly requires `getElementById`. They are not in conflict — they are two
> assignments practising two different APIs, and each is graded on its own text.
> In real code either is fine; `querySelector` is more flexible,
> `getElementById` is marginally faster and clearer when you genuinely have an id.

### Step 2 — listening

```js
greetButton.addEventListener("click", function () { ... });
```

The function is registered, not run. The browser stores it and calls it on each
click.

### Step 3 — capturing and calling

```js
var enteredName = nameInput.value;
var enteredTime = timeInput.value;
var greeting = generateCustomGreeting(enteredName, enteredTime);
outputParagraph.textContent = greeting;
```

`.value` is read **inside** the callback, not outside. This matters: read at
load time, it would capture whatever was in the box then — the empty string —
and every click would produce the same stale greeting.

`.textContent` sets the text of the paragraph. The alternative, `.innerHTML`,
would parse the string as HTML — so a user typing `<b>Bo</b>` would get bold
text, and a user typing a `<script>` tag would be attempting an injection.
`textContent` treats input as text, always. It is both the safer and the more
honest choice when you are writing text.

---

## 3. The interesting part: what "no time provided" means

The prompt says: *"if no time is provided, it should default to `day`."*

The obvious implementation is a **default parameter**:

```js
function generateCustomGreeting(name, time = "day") {   // looks right
  return "Good " + time + ", " + name + "! ...";
}
```

This is wrong here, and the reason is worth internalising.

A default parameter fires when the argument is `undefined`. But the value
arriving from the page is:

```js
timeInput.value   // an empty text input gives ""  ← a string, not undefined
```

`""` is not `undefined`, so the default never fires, and the user sees:

```
Good , Bo! Welcome back to JavaScript class.
```

An empty input field is *the* way a user "provides no time", so the natural
reading of the requirement is precisely the case the natural implementation
misses. The shipped version tests the value:

```js
var timeOfDay = time;

if (timeOfDay === undefined || timeOfDay === null || String(timeOfDay).trim() === "") {
  timeOfDay = "day";
}
```

Three conditions, each earning its place:

- `undefined` — the function called with one argument, e.g. from other code
- `null` — defensive; some APIs hand back `null` for "no value"
- `.trim() === ""` — empty, **or nothing but whitespace**. Someone who types a
  space has still not provided a time.

`String(...)` guards the case where a non-string is passed, so `.trim()` cannot
throw.

This is the assignment's real trap, and it is exactly the kind of thing "audit
the AI's output" is meant to catch: the code looks correct, satisfies the prompt
read literally, and fails on the most ordinary input.

---

## 4. Alternatives considered

**Template literals.**

```js
return `Good ${timeOfDay}, ${name}! Welcome back to JavaScript class.`;
```

Cleaner than `+` concatenation and what most modern code uses. String
concatenation is what the course has covered, so that ships. This is a
readability preference, not a correctness one.

**A ternary instead of `if/else`.**

```js
var timeOfDay = (time && time.trim()) ? time : "day";
```

Shorter. Rejected because the course teaches `if/else`, and the explicit three-
condition form documents *why* each case exists — which is the whole point here.

**Defaulting a blank name too.** The prompt specifies a default only for `time`,
so a blank name produces `"Good day, ! Welcome back..."`. Handling it would be a
one-line change and arguably better UX, but it would deviate from the specified
behaviour. Left literal and flagged in `AUDIT-NOTES.md`.

**Normalising capitalisation.** A user typing `Morning` gets
`"Good Morning, ..."`. Adding `.toLowerCase()` would tidy that, but the prompt
gives an exact output template with no normalisation, so nothing is applied.

**Moving the script into a separate `.js` file.** Better practice generally, and
what the previous assignment required. Here Step 2 says to paste the function
"directly inside the `<script>` tags at the bottom of your HTML file", so it
stays inline.
