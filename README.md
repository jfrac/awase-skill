<div align="center">

# 🥋awase

</div>

>*awase (合わせ) — Japanese for "bringing together" or "matching"; the act of combining elements into a coherent whole.*


You work with AI agents all day. The agent writes the code; you stay sharp by understanding it.
`/awase` turns the code from your session into short exercises — personalized, spaced, and done in under 2 minutes.

---

## How it works

1. Work with Claude Code normally
2. Type `/awase` at any point
3. The agent analyzes the code from the session
4. Generates 1-2 short exercises (< 2 minutes total)
5. Gives immediate feedback
6. Updates your personal profile

Exercises come in five types: **compare** two approaches, **complete** a snippet, **find a bug**, **explain** a block, or answer a **theory** question. The type is chosen based on the concept and your history with it.

## Example session

```
> /awase

┌─ Exercise ──────────────────────────────────────────────────────────┐
│  Promise.all vs Promise.allSettled                                  │
└─────────────────────────────────────────────────────────────────────┘

Which is more appropriate for the processUsers function we just wrote?

  A)  const results = await Promise.all(ids.map(fetchUser));

  B)  const results = await Promise.allSettled(ids.map(fetchUser));

> B, because fetchUser can fail and I don't want it to cancel the rest

✓ Correct. Promise.allSettled waits for all promises regardless of
  individual failures — ideal when you want partial results instead
  of a total abort.

  Next review of Promise.allSettled in 6 days.
```

---

## Installation

### Option A — Personal skill (simplest)

```bash
mkdir -p ~/.claude/skills/awase
curl -fsSL https://raw.githubusercontent.com/jfrac/awase-skill/main/skills/awase/SKILL.md \
  > ~/.claude/skills/awase/SKILL.md
```

### Option B — Plugin via marketplace

```
/plugin marketplace add jfrac/awase-skill
/plugin install awase@jfrac/awase-skill
```

`/awase` becomes available in all your projects immediately. The profile is created automatically on first use.

---

## Usage

| Command | Description |
|---|---|
| `/awase` | Normal flow — agent picks the exercise |
| `/awase --tipo compare` | Force a comparison exercise |
| `/awase --tipo completar` | Force a snippet completion |
| `/awase --tipo bug` | Force a find-the-bug exercise |
| `/awase --tipo explicar` | Force an explain-the-code exercise |
| `/awase status` | Show your profile: concepts, hit rate, upcoming reviews |
| `/awase skip` | Skip the session without touching the profile |
| `/awase reset` | Reset the full profile (asks for confirmation) |

---

## Spaced Repetition

awase uses the **SM-2** algorithm (the same one behind Anki) to schedule reviews. Concepts you know well are reviewed less often; ones you struggle with come back sooner. Over time, the agent builds a precise picture of what you need to revisit and when.

Your profile lives at `~/.awase/profile.json` — on your machine, never in the repo.
