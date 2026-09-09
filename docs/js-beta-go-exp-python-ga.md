# JS `beta`, Go `exp`, and what Python will ship

JS `import { genkit } from 'genkit/beta'` is an import channel, not a quality rating. It was stood up in Jan 2025 to hold Chat/Session. Other APIs got swept in. Almost nothing was ever taken *off* the beta class after it also landed on stable — so “still in beta” does not mean “still unfinished.”

I split the JS list into: already on stable (but never cleaned out of beta), still beta-only, and deleted. For each, where Go puts it, and what Python will do.

## Already on JS stable — leftover on `beta`

These work if you imported ordinary `genkit`. They also still appear on the beta class / export path.

- **Built-in formats** (`json`, `text`, `enum`, … on `generate`) — stable in JS and Go. **Python: stable** (already on `from genkit import Genkit`).
- **`checkOperation` / `cancelOperation`** — stable in JS. Go: check is stable; cancel is on the model action. **Python: stable.**
- **`generate({ resume })`** — stable in JS and Go (no beta gate). Only useful once something can interrupt. **Python: stable.**
- **`respond` / `restart` helpers** — exported from stable JS `genkit` with no runtime gate. Go: stable `RespondWith` / `RestartWith`. **Python: stable** (same leftover: tool pauses, caller resumes).
- **`defineBackgroundModel`** — stable in JS and Go. **Python: stable.**
- **`currentSession()`** — on JS stable with a `@beta` docstring only; no throw. **Python: stays on the stable class** so a tool does not have to import `genkit.exp` to read the session.

The leak: JS still *labels* interrupt-in-a-tool and `generateOperation` as beta (`assertUnstable`), but resume, poll, and cancel already sit on stable.

## Still JS-beta-only (never graduated)

**Interrupts** (`defineInterrupt`, `interrupt()` inside a tool, `tool.respond` / `tool.restart` as the gated path)

Sat here ~20 months (Jan 2025). The leftover has not moved: tool pauses, caller gets the interrupt, they resume or restart.

**Go: stable** (`InterruptWith` on ordinary tools). There is also a newer typed `DefineInterruptibleTool` in `genkit/exp` that they say will replace the old shape in a major.

**Python: stable.** Ship on the main class. Do not make people import `genkit.exp` to pause a tool.

**Custom formats** (`defineFormat`)

Parked on the JS beta class Feb 2025. The formats themselves have been on stable `generate` since late 2024.

**Go: stable** (`DefineFormats`).

**Python: stable.**

**Start a long-running job** (`generateOperation`)

JS: starting the job is beta-only (Jun 2025). Checking and canceling the ticket are already stable.

**Go: stable** (`GenerateOperation`).

**Python: stable** — start, check, and cancel all on the main class.

**Resources** (`defineResource`)

JS: beta class only (Jun 2025). Never had a runtime gate.

**Go: stable** (`DefineResource`).

**Python: drop.** Do not ship resources as part of Python GA.

**Agents** (`defineAgent` / `definePromptAgent` / `defineCustomAgent`)

JS: beta class only. Landed 24 Jun 2026 — same day JS deleted the old `ai.chat()` / create-session API. ~2.5 months old.

**Go: experimental** (`genkit/exp` + `WithExperimental()`).

**Python: experimental.** `from genkit.exp import Genkit` is how you opt in. `from genkit import Genkit` has no `define_agent`. Types still live on `genkit.agent`. One exp instance still runs `generate`.

**Old Chat / Session (`ai.chat()` on the app)**

JS: **deleted** from stable (Jan 2025); agents are the replacement, still in beta.

**Go: never had that client.** Agents are HTTP turns + session/snapshot ids.

**Python: do not resurrect `ai.chat()`. Agents are the product, behind `genkit.exp`.**

## JS-beta leftovers we are not inventing for Python

- **`runFlow` / `streamFlow`** (`genkit/beta/client`) — old flow HTTP helpers, moved under beta in Feb 2025. Go has no named equivalent. Python is not adding a beta client just to match.
- **Durable `StreamManager`** (Dec 2025) — JS experiment; Go already has a stable in-memory stream manager plus Firebase exp stores. Python does not have this and we are not adding it for GA.
- **Firebase Data Connect** (`@genkit-ai/firebase/beta`) — JS only. Go does not have it. Python is not adding it.

Firestore session store is different: it is how you persist an **agent** session. JS/Go keep it next to agents (plugin `/beta` or `exp`). Python keeps `FirestoreSessionStore` on `genkit-google-cloud` for apps that opted into `genkit.exp`.

## Python in one line

Graduate on whether the caller program still churns, not on the JS label. Interrupts, formats, long-running generate, resume, and check/cancel ship on the main `Genkit`. Agents stay behind `genkit.exp`. Resources we drop. Old chat, StreamManager, Data Connect, and the flow HTTP client we do not add.

That is how Python can be generally available without implying agents are done, and without dragging JS’s leftover parking lot into the default import.
