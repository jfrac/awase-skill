# awase-skill Evaluation Tests

This directory contains test cases for the awase skill using the skill-creator eval framework.

## Overview

- **8 critical test cases** covering main flows, edge cases, subcommands, and SM-2 algorithm
- **Objective expectations** based on verifiable profile fields and SM-2 calculations
- **Manual execution** recommended (avoid CI due to API costs)

## Test Coverage

| ID | Test Case | What It Validates |
|----|-----------|-------------------|
| 1 | Happy path - new concept | Concept extraction, exercise generation, profile creation |
| 2 | Empty session edge case | Graceful handling when no code available |
| 3 | Pending review priority | Concept selection logic (priority 1: pending in session) |
| 4 | Forced type with --tipo | Type override mechanism |
| 5 | Status subcommand | Profile statistics display and calculation |
| 6 | Skip subcommand | No-op behavior without side effects |
| 7 | Reset confirmation | Safety confirmation before destructive action |
| 8 | SM-2 algorithm correctness | Interval, repetitions, easiness_factor updates |

## How to Run Evals

### Prerequisites

1. **Install skill-creator**:
   ```bash
   claude
   # In Claude Code session:
   /plugin marketplace add anthropics/skill-creator
   /plugin install skill-creator@anthropics/skill-creator
   ```

2. **Install awase skill** (if not already installed):
   ```bash
   # Copy to Claude skills directory
   cp -r skills/awase ~/.claude/skills/
   ```

### Running Evals Manually

**Step 1: Start Claude Code**
```bash
claude
```

**Step 2: Invoke skill-creator**
```
/skill-creator
```

**Step 3: Run eval mode**
```
Evaluate the awase skill using the evals at ~/.claude/skills/awase/evals/evals.json
```

**Step 4: Review results**

Skill-creator will:
- Execute each test case multiple times
- Generate `grading.json` with pass/fail results
- Create HTML viewer for interactive results
- Calculate pass rate and identify flaky tests

### Understanding Results

**Grading Output** (`evals/grading.json`):
```json
{
  "eval_id": 1,
  "passed": true,
  "expectations": [
    {
      "text": "Exercise is generated with one of 5 valid types",
      "passed": true,
      "evidence": "Exercise type is 'completar'"
    }
  ]
}
```

**Key Metrics:**
- **Pass rate**: Percentage of expectations that passed
- **Flaky tests**: High variance across runs (investigate these)
- **Evidence**: Specific output citations for each expectation

## Fixtures

### Profile Fixtures

Located in `fixtures/profiles/`:

- **empty.json**: First-time user, no concepts trained
- **with-pending.json**: 2 concepts due for review today (2026-05-09)
- **with-history.json**: Extensive history with 3 concepts in different states

### Session Code Examples

Located in `fixtures/session-examples/`:

- **redis-cache.md**: Caching implementation (Layer 1 + Layer 2 concepts)
- **promises.md**: Promise error handling patterns
- **database-tx.md**: SQL transactions with ACID properties

Use these to simulate realistic sessions during testing.

## Expectations Philosophy

### ✅ Good Expectations (Objective, Non-Flaky)

- Profile field values: `interval = 1`, `repetitions = 0`
- File existence: "Profile exists at ~/.awase/profile.json"
- Enum matching: "Exercise type is one of 5 valid types"
- Mathematical validation: "next_review = today + 1 day"
- String patterns: "Output contains 'Session skipped'"

### ❌ Avoid (Subjective, Flaky)

- Code quality: "Code is well-written"
- Exact text matching: "Output is exactly 'foo bar'"
- Token counts: "Uses exactly 150 tokens"
- Exercise content quality: "Question is good"

## When to Run Evals

**Recommended triggers:**
- ✅ Before releasing new version
- ✅ After significant SKILL.md changes
- ✅ When modifying SM-2 algorithm logic
- ✅ Weekly/monthly benchmarks

**Avoid:**
- ❌ On every git commit (expensive)
- ❌ Automated CI/CD (API costs accumulate)
- ❌ Minor documentation changes

## Cost Considerations

**Per eval run**: ~$0.04-0.50 USD depending on:
- Number of test cases (8 in this set)
- Iterations per case (skill-creator runs multiple times)
- Model used (Sonnet 4.5 recommended)

**Monthly budget estimate**:
- 4 runs/month × $0.30/run = **~$1.20/month** (with prompt caching)
- 10 runs/month × $0.50/run = **~$5/month** (without caching)

## Benchmark Mode (After Evals)

Once evals are passing, run benchmark mode for statistical analysis:

```
/skill-creator

Run benchmark mode on the awase skill using the evals. Execute 10 iterations to measure variance.
```

This generates:
- Mean ± stddev for each metric
- Flaky test detection (high variance)
- Token usage analysis
- Performance comparison

## Next Steps

1. **Run initial evals** - Execute all 8 test cases
2. **Review grading.json** - Check which expectations pass/fail
3. **Iterate on SKILL.md** - Fix failing tests
4. **Add more tests** - Cover additional edge cases discovered
5. **Benchmark periodically** - Measure consistency improvements

## Questions?

See the main [README.md](../README.md) for installation and usage instructions, or check the [skill-creator documentation](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/skill-creator).
