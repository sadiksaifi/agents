# Agent Skills Authoring Guide

This repository is a collection of agent skills. This file governs creating, editing, and maintaining skills here.

## Source of Truth

When guidance conflicts, use this precedence:

1. [Agent Skills specification](https://agentskills.io/specification) (normative)
2. This file (repository rules)
3. External guides (advisory)

## Skill Structure

Each skill lives in `skills/<kebab-name>/` with one required file:

```
skills/<kebab-name>/
├── SKILL.md              # Required — instructions loaded when triggered
├── references/           # Optional — docs loaded on demand
├── scripts/              # Optional — executable utilities
└── assets/               # Optional — templates, static files
```

`<kebab-name>` must match the `name` field in SKILL.md frontmatter.

Do not add auxiliary files (README, CHANGELOG) inside skill folders.

## SKILL.md Format

### Frontmatter

```yaml
---
name: skill-name
description: Processes X and generates Y. Use when working with X or when the user mentions Z.
---
```

**Required fields:**

| Field | Constraints |
|-------|-------------|
| `name` | 1–64 chars. Lowercase alphanumeric and hyphens only. Must not start/end with `-`. No consecutive hyphens (`--`). No reserved words (`anthropic`, `claude`). Must match folder name. |
| `description` | 1–1024 chars. Describes what the skill does AND when to use it. Third person ("Processes files..." not "I can help you..." or "You can use this to..."). Include specific keywords that help agents match tasks. |

**Optional fields:**

| Field | Purpose |
|-------|---------|
| `license` | License name or reference to bundled file. |
| `compatibility` | Max 500 chars. Environment requirements (runtime, packages, network). |
| `metadata` | Arbitrary key-value pairs (e.g., `author`, `version`). |
| `allowed-tools` | Space-delimited list of pre-approved tools. Experimental. |

### Body

No format restrictions — write whatever helps agents perform the task. The full body loads when the skill activates, so every token competes with conversation history.

**Size target:** Keep the body under 500 lines. Move detailed reference material to `references/`.

### References

- Link directly from SKILL.md — one level deep only. Avoid nested reference chains.
- One concept per file.
- Use relative paths: `[guide](references/guide.md)`.

## Authoring Principles

### Conciseness

The agent is already smart. Only add context it doesn't have. Challenge every paragraph: does it justify its token cost?

### Progressive disclosure

Three tiers of context loading:

1. **Metadata** (~100 tokens) — `name` and `description` load at startup for all skills.
2. **Instructions** (<5000 tokens recommended) — SKILL.md body loads when triggered.
3. **Resources** (as needed) — files in `references/`, `scripts/`, `assets/` load on demand.

### Degrees of freedom

Match specificity to fragility:

- **High freedom** (text-based guidance): multiple valid approaches, context-dependent decisions.
- **Medium freedom** (pseudocode, parameterized scripts): a preferred pattern exists with acceptable variation.
- **Low freedom** (exact scripts, no parameters): fragile operations where consistency is critical.

### Description quality

The description drives skill selection from potentially 100+ available skills. It must provide enough signal for the agent to match the right skill to the current task.

### No time-sensitive information

Avoid dates, version cutoffs, or "before/after" language. Use an "old patterns" section with `<details>` for deprecated approaches.

### Consistent terminology

Pick one term for each concept and use it throughout. Don't alternate between synonyms (e.g., "field" vs "box" vs "element").

## Authoring Workflow

1. Use the `skill-creator` skill for guidance before drafting.
2. Define target trigger scenarios and non-goals.
3. Write `SKILL.md` with minimal, high-signal instructions.
4. Add `references/`, `scripts/`, or `assets/` only if they reduce repetition or improve reliability.
5. Run the validation checklist below.
6. Refine for concision and trigger precision.

## Validation Checklist

### Structure

- [ ] Skill folder path is `skills/<kebab-name>/`.
- [ ] `SKILL.md` exists in the skill folder.
- [ ] Optional resources are under `references/`, `scripts/`, or `assets/` only.

### Frontmatter

- [ ] `name` exists, matches folder name, follows naming constraints.
- [ ] `description` exists, describes triggers, uses third person.
- [ ] No placeholder values remain.

### Content

- [ ] Clear when-to-use guidance present.
- [ ] Workflow is actionable and scoped.
- [ ] No unnecessary repetition or filler.
- [ ] Deep details moved to references where appropriate.

### Links

- [ ] Relative links from SKILL.md resolve.
- [ ] All referenced files exist.

### Suggested commands

```sh
fd -t f SKILL.md skills
fd -t f . skills/*/references
rg -n "^---$|^name:|^description:" skills/*/SKILL.md
```

## Authoritative References

- Agent Skills home: <https://agentskills.io/home>
- What are skills: <https://agentskills.io/what-are-skills>
- Agent Skills specification: <https://agentskills.io/specification>
- Integrate skills: <https://agentskills.io/integrate-skills>
- Anthropic overview: <https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview>
- Anthropic best practices: <https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices>
