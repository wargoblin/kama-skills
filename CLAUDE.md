# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Community-driven collection of **skills** for Claude Code — drop-in markdown instruction modules that give Claude new capabilities (presentation generation, project scaffolding, complex workflows, etc.).

## Repository Structure

```
kama-skills/
  <skill-name>/                    # Each skill gets its own top-level directory
    .claude/skills/<skill-name>/   # The actual skill (tracked by git)
      SKILL.md                     # Entry point — full procedure Claude follows
      references/                  # Supporting docs, syntax guides, patterns
      assets/                      # Templates, demo files
    .gitignore                     # Allowlist pattern — ignores everything except the skill
    README.md                      # Skill test area docs
```

Each skill directory serves dual purpose: **skill source** (in `.claude/skills/`) and **test playground** (everything else is gitignored via allowlist pattern).

## Current Skills

| Skill | Directory | Trigger |
|-------|-----------|---------|
| Slidev Presentation Generator | `slidev/` | `/slidev` |
| Outline Structure Generator | `outline/` | `/outline` |
| Game Coach (Gamified Goals) | `game-coach/` | `/game` |
| Slidegen Image Presenter | `slidegen/` | `/slidegen` |
| SlideCraft (Two-Layer Presentations) | `slidecraft/` | `/slidecraft` |

## Working with Skills

### Committing skill changes only
```bash
# Only .claude/ contents are tracked — experiments are gitignored
git add slidev/.claude/
git commit -m "fix(slidev): description"
```

### Adding a new skill
1. Create `skills/<name>/.claude/skills/<name>/SKILL.md`
2. Add an allowlist `.gitignore` in `skills/<name>/` (copy from `slidev/.gitignore`)
3. Add references and assets as needed
4. Register in `.claude/settings.json` with path, description, and trigger

### Skill quality expectations
- **Self-contained**: all instructions in SKILL.md + references, no implicit knowledge
- **Testable**: produces observable, verifiable output
- **Robust**: handles errors, validates inputs, has fallback strategies
- Mark critical rules with `**CRITICAL**` or `MUST` — Claude respects emphasis hierarchy
- Keep SKILL.md under ~4000 lines; split into references/ if larger

## Autonomous Execution

**CRITICAL:** When executing plans, DO NOT stop to ask for confirmation between steps.
Execute all steps autonomously without interruption.
If you encounter a blocker, attempt to resolve it yourself (2-3 attempts).
Only stop if the blocker is truly unresolvable after multiple attempts.
Do not ask "should I continue?" — just continue.
Do not present options for the next step — pick the best one and execute.
