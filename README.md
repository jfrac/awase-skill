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

Exercises come in five types: **compare** two code options, **complete** a snippet, **find a bug**, **explain** what a block does, or **theory** questions about underlying concepts. The agent picks the most appropriate type based on your history.

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

## Testing & Validation

This skill includes a comprehensive test suite using the skill-creator eval framework.

### Running Tests Manually

```bash
# 1. Start Claude Code
claude

# 2. In the session, invoke skill-creator
/skill-creator

# 3. Run evals
Evaluate the awase skill using the evals at ~/.claude/skills/awase/evals/evals.json
```

**8 test cases** cover:
- New concept extraction and exercise generation
- Pending review priority selection
- SM-2 algorithm correctness (interval, repetitions, easiness_factor)
- Subcommands (status, skip, reset)
- Edge cases (empty session, forced type)

See [evals/README.md](evals/README.md) for detailed instructions and expectations.

### Automated Validation (CI)

GitHub Actions validates schema and structure on each PR:
- ✅ JSON schema validation (`profile.schema.json`)
- ✅ YAML frontmatter validation (`SKILL.md`)
- ✅ File structure checks

**Note**: Full evals run manually (not in CI) to avoid API costs.

### Example Test Scenarios

**Scenario 1: New Concept from Redis Session**
```
User works on Redis caching code
→ /awase extracts concepts: redis.get(), redis.set(), TTL, cache eviction
→ Generates completar or compare exercise for Layer 1 concept
→ Profile created with SM-2 defaults (easiness=2.5, interval=1, reps=0)
```

**Scenario 2: Pending Review Priority**
```
User has 2 concepts due today
→ /awase prioritizes concept from current session
→ Exercise type respects last_exercise_type (no consecutive repeats)
→ SM-2 updates interval based on response quality
```

**Scenario 3: Theory Exercise (Layer 2)**
```
User writes database transaction code
→ /awase extracts underlying concept: "transaction isolation levels"
→ Generates theory question (no code required)
→ "What is the difference between optimistic and pessimistic locking?"
```

See `evals/fixtures/session-examples/` for complete session code samples.

---

## Repo structure

```
awase-skill/
  .claude-plugin/
    plugin.json                      plugin manifest
  skills/
    awase/
      SKILL.md                       agent instructions
  evals/
    evals.json                       test cases for skill-creator
    README.md                        testing instructions
    fixtures/
      profiles/                      example profile states
      session-examples/              sample session code
  profile.schema.json                personal profile structure
  README.md                          this file
  CLAUDE.md                          guidance for Claude Code
```

The dev profile is **not** in the repo. It is stored locally at `~/.awase/profile.json`.
