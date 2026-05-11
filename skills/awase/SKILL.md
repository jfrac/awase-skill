---
description: Adaptive training skill for developers. Generates a short technical exercise based on session code and underlying concepts, and maintains a personal profile with spaced repetition (SM-2). Use when the user writes '/awase', '/awase status', '/awase skip', '/awase reset', or '/awase --tipo <type>'.
argument-hint: "[--tipo <type> | status | skip | reset]"
allowed-tools: Read Write Bash(mkdir *) Bash(rm ~/.awase/profile.json)
---

# awase — Adaptive developer training skill

This skill activates when the user writes `/awase` (with or without arguments).
It generates short technical exercises — both code-based and purely conceptual — derived
from the current session, and maintains a personal profile using spaced repetition (SM-2 algorithm).

---

## Subcommands

| Invocation | Behavior |
|---|---|
| `/awase` | Normal flow: agent chooses exercise type |
| `/awase --tipo compare` | Force comparison exercise |
| `/awase --tipo completar` | Force snippet completion exercise |
| `/awase --tipo bug` | Force find-the-bug exercise |
| `/awase --tipo explicar` | Force explain-the-code exercise |
| `/awase --tipo theory` | Force pure theory question |
| `/awase status` | Show the dev's profile |
| `/awase skip` | Skip the session without penalizing the profile |
| `/awase reset` | Reset the profile (asks for confirmation) |

---

## Personal profile

**Path:** `~/.awase/profile.json`

If the file does not exist, create it before continuing with this initial structure:

```json
{
  "version": "1",
  "created_at": "<today ISO 8601>",
  "last_session": null,
  "consecutive_code_exercises": 0,
  "concepts": {},
  "sessions": []
}
```

Always use the `Read` and `Write` tools to read and write the profile.
Never mention the file path or the raw JSON to the user; use natural language.

---

## Normal flow

### Step 1 — Read the profile
Read `~/.awase/profile.json`. Create it if it doesn't exist.

### Step 2 — Identify trainable concepts
Examine the session and extract two layers of concepts:

**Layer 1 — Surface concepts** (directly visible in the code): specific APIs, library
functions, language features, data structures, and patterns that appear in the session.

**Layer 2 — Underlying concepts** (implied by the work done): CS fundamentals, system
design trade-offs, protocol behaviour, algorithmic complexity, architectural decisions,
and domain knowledge that the session touched on but didn't make explicit. For example,
if the session wrote a Redis cache, Layer 2 includes cache eviction strategies and
consistency guarantees — even if no code about them was written.

Discard trivial or overly basic concepts for the dev's level.
Keep 3-5 candidates total across both layers.

### Step 3 — Select concept to drill
Priority:

1. Session concepts with `next_review <= today` in the profile (pending review)
2. Session concepts not yet in the profile (new)
3. Any profile concept with `next_review <= today` (generic review)

If no concept is available, tell the user briefly.

### Step 4 — Choose exercise type
If the user forced a type with `--tipo`, use it.

If `consecutive_code_exercises >= 3` in the profile, force `theory` regardless of concept state.

Otherwise, choose based on the concept's history and origin:

| Concept state | Preferred types |
|---|---|
| New (no history), Layer 1 | `completar`, `compare` |
| New (no history), Layer 2 | `theory`, `compare` |
| 1-2 correct repetitions | `bug`, `explicar`, `theory` |
| 3+ correct repetitions | any, prefer `theory` or `explicar` |

Never repeat the same type as `last_exercise_type` for that concept.

### Step 5 — Generate the exercise

Use real session code for code-based types. For `theory`, no code is needed.
The full exercise must be solvable in under 2 minutes.

**`compare`** — two code versions, user picks the most appropriate and explains why:

```
**Exercise — [concept]**

Which is more appropriate for [brief context]?

A)
[code A]

B)
[code B]
```

**`completar`** — snippet with `___` gaps the user must fill in:

```
**Exercise — [concept]**

Complete the following code:

[code with ___]
```

**`bug`** — snippet with a subtle bug; user must find and explain it:

```
**Exercise — [concept]**

There is a bug in this code. Can you find it?

[code with bug]
```

**`explicar`** — code block; user explains what it does and why:

```
**Exercise — [concept]**

Explain what this code does and why it is written this way:

[code]
```

**`theory`** — a direct conceptual question; no code required. Used for underlying
concepts, system design trade-offs, CS fundamentals, and protocol behaviour:

```
**Exercise — [concept]**

[Direct question about the concept. One or two sentences.]
```

Examples of good theory questions:
- "What happens when a Redis key expires and a read request arrives at the same time?"
- "What is the difference between optimistic and pessimistic locking, and when would you choose each?"
- "Why does TCP guarantee order but UDP does not, and what cost does that come at?"

### Step 6 — Wait for user response
Present the exercise and wait. Do not proceed until a response is received.

### Step 7 — Evaluate the response

Map response quality to the SM-2 scale:

| q | Criterion |
|---|---|
| 5 | Correct with solid explanation |
| 4 | Correct |
| 3 | Correct but hesitant or incomplete explanation |
| 1 | Incorrect but user recognizes the error after feedback |
| 0 | Incorrect without understanding |

### Step 8 — Apply SM-2 and update profile

```
if q < 3:
    interval = 1
    repetitions = 0
if q >= 3:
    if repetitions == 0: interval = 1
    if repetitions == 1: interval = 6
    if repetitions > 1: interval = round(interval * easiness_factor)
    easiness_factor = max(1.3, easiness_factor + 0.1 - (5-q) * (0.08 + (5-q) * 0.02))
    repetitions += 1

next_review = today + interval days
```

Default values for a new concept: `easiness_factor=2.5`, `interval=1`, `repetitions=0`.

Update the concept in `concepts` and add an entry to the `sessions` array:
```json
{
  "date": "<today>",
  "exercises": 1,
  "correct": <1 if q>=3, else 0>,
  "concepts_trained": ["<concept_id>"]
}
```

If there is already an entry for today in `sessions`, accumulate instead of adding a new one.

Update `last_session` with today's date.

If the exercise type was `theory`, set `consecutive_code_exercises = 0`. Otherwise, increment it by 1.

Write the profile with `Write`.

### Step 9 — Show feedback

Show in three parts:
1. `✓ Correct.` or `✗ Incorrect.` followed by a brief concept explanation (3 lines max)
2. If incorrect, one line with the correct answer
3. `Next review of [concept] in N days.`

---

## Subcommand: status

Read the profile and display:

```
**awase profile**

Concepts trained: N
Total sessions: N
Global hit rate: N%

**Upcoming reviews:**
[concept]        due [date]    current interval: N days

**New concepts available in this session:**
[concept]
```

If the profile is empty, indicate there is no data yet.

---

## Subcommand: skip

Reply with a single line: `Session skipped. Profile unchanged.`
Do not read or write the profile.

---

## Subcommand: reset

Ask for explicit confirmation before acting:

```
Are you sure you want to delete the entire profile? Type "confirm" to continue.
```

If the user confirms, delete `~/.awase/profile.json` using `Bash` (`rm ~/.awase/profile.json`)
and reply: `Profile reset.`

If the user does not confirm or responds with anything else, reply: `Cancelled.`

---

## General rules

- An exercise must be solvable in under 2 minutes. Shorten if needed.
- For code-based types, use real session code when possible; only invent code as a last resort.
- For `theory`, the question must be precise and answerable in 2-4 sentences.
- Feedback is always concise: 3 lines of explanation maximum.
- Do not drill trivial concepts (e.g. basic syntax the dev clearly masters).
- Never mention the JSON file or the SM-2 algorithm to the user; speak of "profile" and "next review".
- If the session has no relevant code or concepts, say so briefly and offer `/awase skip`.
