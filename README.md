# Agent Skills

Opinionated agent skills for software development — because even smart models need a good playbook.

## Skills

### Ladder

Phased, spec-driven software development using the **Ladder** methodology.

| Skill | Description |
|-------|-------------|
| [`ladder-init`](skills/ladder-init/SKILL.md) | Initializes Ladder project structure with OVERVIEW.md and progress tracking |
| [`ladder-create`](skills/ladder-create/SKILL.md) | Creates new phase specs in canonical format with step sequences and acceptance criteria |
| [`ladder-execute`](skills/ladder-execute/SKILL.md) | Executes a Ladder phase spec step-by-step with commit discipline and progress tracking |
| [`ladder-refine`](skills/ladder-refine/SKILL.md) | Rewrites a phase spec into canonical Ladder format without altering intent |

### Installation

```sh
npx skills add sadiksaifi/agents
```

### Ladder Workflow

```
/ladder-init → /ladder-create → /ladder-refine (if needed) → /ladder-execute
```

```
┌──────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ /ladder-init │────▶│ /ladder-create   │────▶│ /ladder-execute │
│              │     │                  │     │                 │
│ Creates:     │     │ Creates:         │     │ Reads:          │
│ OVERVIEW.md  │     │ L-<N>-slug.md    │     │ spec + progress │
│ progress.md  │     │ Updates:         │     │ Updates:        │
│              │     │ OVERVIEW.md      │     │ progress.md     │
└──────────────┘     └────────┬────────┘     └─────────────────┘
                              │                      ▲
                              │ validation fails     │
                              ▼                      │
                     ┌─────────────────┐             │
                     │ /ladder-refine  │─────────────┘
                     │                 │
                     │ Rewrites:       │
                     │ spec in-place   │
                     └─────────────────┘
```

1. **`/ladder-init`** — Bootstrap the `.ladder/` directory (once per project)
2. **`/ladder-create`** — Create phase specs for planned work
3. **`/ladder-refine`** — Fix any specs that need restructuring
4. **`/ladder-execute`** — Implement phases step-by-step with progress tracking

### Compatibility

Compatible with any agent supporting the [Agent Skills specification](https://agentskills.io/specification) — including Claude Code, Codex, Gemini CLI, and others. Distributed through [skills.sh](https://skills.sh).

## License

[MIT](LICENSE)
