# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **awase-skill**, a Claude Code plugin that provides adaptive developer training through spaced repetition. The plugin generates short technical exercises (1-2 minutes) based on code written during Claude Code sessions, maintaining a personal SM-2-based profile at `~/.awase/profile.json`.

The skill is invoked by users with `/awase` and operates entirely through natural language interaction - there is no build process, test suite, or code execution in this repository.

## Repository Structure

```
awase-skill/
  .claude-plugin/
    plugin.json              Plugin manifest (name, description, author)
  skills/
    awase/
      SKILL.md              Complete agent instructions for /awase skill
  profile.schema.json       JSON Schema for user profile structure
  README.md                 User-facing documentation
```

## Key Architecture

### Plugin System
- This is a **Claude Code plugin**, not a traditional software project
- The plugin manifest is at `.claude-plugin/plugin.json`
- The skill definition is at `skills/awase/SKILL.md` with YAML frontmatter
- No executable code exists in this repository - everything is instruction-based

### Personal Profile Storage
- User profiles are stored at `~/.awase/profile.json` (NOT in this repo)
- Profile structure is defined in `profile.schema.json`
- Uses SM-2 spaced repetition algorithm to schedule concept reviews
- Concepts have: interval, easiness_factor, repetitions, next_review, history

### Two-Layer Concept Extraction
The skill extracts concepts from sessions at two levels:
1. **Layer 1 (Surface)**: Direct code elements (APIs, functions, language features)
2. **Layer 2 (Underlying)**: Implied fundamentals (CS concepts, trade-offs, protocols, architecture decisions)

### Exercise Types
Five exercise types with specific selection logic:
- `compare`: Choose between two code approaches
- `completar` (complete): Fill in code gaps
- `bug`: Find and explain a subtle bug
- `explicar` (explain): Explain code purpose and rationale
- `theory`: Pure conceptual questions (no code required)

Exercise selection depends on:
- Concept state (new vs. reviewed)
- Layer origin (1 vs. 2)
- Repetition count
- Previous exercise type (never repeat consecutively for same concept)

### SM-2 Implementation
The skill implements the SM-2 algorithm with:
- Quality scale 0-5 for response evaluation
- Interval calculation based on repetitions and easiness factor
- Automatic interval reset on quality < 3
- Default starting values: easiness_factor=2.5, interval=1, repetitions=0

## Working with This Repository

### Editing the Skill
The primary file to edit is `skills/awase/SKILL.md`. This contains:
- YAML frontmatter with skill description (used for skill triggering)
- Complete instructions for how Claude should behave when `/awase` is invoked
- Subcommand specifications (status, skip, reset)
- Exercise generation logic and templates
- SM-2 algorithm implementation details

### Editing Documentation
- `README.md`: User-facing installation and usage instructions
- English is the primary language (repo was translated from Spanish)

### Plugin Metadata
Edit `.claude-plugin/plugin.json` to update:
- Plugin name
- Description (shown in marketplace)
- Author information

### Profile Schema
`profile.schema.json` defines the structure of `~/.awase/profile.json`. Changes here should be coordinated with SKILL.md logic.

## Common Modifications

### Adding a New Exercise Type
1. Add to the `enum` in `profile.schema.json` (lines 77, 107)
2. Update exercise selection logic in SKILL.md (Step 4)
3. Add exercise template in SKILL.md (Step 5)
4. Update README.md usage section

### Modifying SM-2 Parameters
The SM-2 algorithm is implemented in SKILL.md Step 8. Key parameters:
- `easiness_factor` initial: 2.5, minimum: 1.3
- Interval progression: 1 day → 6 days → interval * easiness_factor
- Quality mapping: 0-5 scale defined in Step 7

### Changing Concept Selection Priority
Priority logic is in SKILL.md Step 3:
1. Pending reviews from current session
2. New concepts from current session
3. Any pending review from profile

## Installation Methods

Users can install via:
1. **Manual**: Copy SKILL.md to `~/.claude/skills/awase/`
2. **Marketplace**: `/plugin marketplace add jfrac/awase-skill` then `/plugin install awase@jfrac/awase-skill`

## Important Constraints

- **No code execution**: This is a pure instruction-based plugin
- **Profile privacy**: User profiles are never committed to repos
- **2-minute rule**: All exercises must be solvable in under 2 minutes
- **Natural language only**: Never expose JSON structure or SM-2 algorithm details to users
- **Balance requirement**: No more than 3 consecutive code-based exercises without a theory exercise
