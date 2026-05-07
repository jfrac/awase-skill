# awase

> `/awase` — adaptive developer training skill for Claude Code

An adaptive training skill for developers working with AI agents. Invoke it mid-session in Claude Code and it generates 1-2 short exercises based on the code the agent just wrote.

The idea is simple: the agent already knows what code you wrote. Instead of ignoring it, `/awase` turns it into personalized training material so you don't lose technical skills through delegation.

---

## How it works

1. Work with Claude Code normally
2. At any point type `/awase`
3. The agent analyzes the code produced in the session
4. Generates 1-2 short exercises (< 2 minutes total)
5. Gives immediate feedback
6. Updates your personal profile with the result

Exercises come in four types: **compare** two code options, **complete** a snippet, **find a bug**, or **explain** what a block does. The agent picks the most appropriate type based on your history.

## Spaced Repetition

The system uses the **SM-2** algorithm (the same one Anki uses) to decide what to drill. Each technical concept has a review interval that lengthens when you get it right and shortens when you don't. Over time, the agent knows exactly what you need to revisit and when.

The profile is **personal**: stored at `~/.awase/profile.json` on your machine, never in the repo.

---

## Installation

### Option A — Personal skill (simplest)

Copy the SKILL.md to your global skills directory:

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

In both cases, `/awase` becomes available in **all your projects** immediately.

### Profile is created automatically

The first time you run `/awase`, the agent creates `~/.awase/profile.json` automatically. Nothing else to do.

Optional: add `~/.awase/` to your global `.gitignore` to make sure the profile is never committed.

```bash
echo "~/.awase/" >> ~/.gitignore_global
git config --global core.excludesfile ~/.gitignore_global
```

---

## Usage

```
/awase                    normal flow, agent decides
/awase --tipo compare     force comparison exercise
/awase --tipo completar   force snippet completion exercise
/awase --tipo bug         force find-the-bug exercise
/awase --tipo explicar    force explain-the-code exercise
/awase status             show your profile: concepts, hit rates, upcoming reviews
/awase skip               skip the session without penalizing the profile
/awase reset              reset the full profile (asks for confirmation)
```

---

## Example session

```
> /awase

**Exercise — `Promise.all` vs `Promise.allSettled`**

Which is more appropriate for the `processUsers` function we just wrote?

A)
const results = await Promise.all(ids.map(fetchUser));

B)
const results = await Promise.allSettled(ids.map(fetchUser));

---

> B, because fetchUser can fail and I don't want it to cancel the rest

✓ Correct. `Promise.allSettled` waits for all promises regardless of whether
they fail, ideal when you want partial results instead of a total failure.

Profile updated. Next review of `Promise.allSettled` in 6 days.
```

---

## Repo structure

```
awase-skill/
  .claude-plugin/
    plugin.json           plugin manifest
  skills/
    awase/
      SKILL.md            agent instructions
  profile.schema.json     personal profile structure
  README.md               this file
```

The dev profile is **not** in the repo. It is stored locally at `~/.awase/profile.json`.
