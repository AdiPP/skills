# Agent Skills

Reusable AI coding agent skills in [Agent Skills](https://agentskills.io) format (YAML frontmatter + Markdown body).

## Available Skills

| Skill | Description |
|-------|-------------|
| [conventional-commits](skills/conventional-commits/) | Git commit message format following Conventional Commits spec |
| [hyperf-backend](skills/hyperf-backend/) | Hyperf microservice code generation with DDD, Clean Architecture, Kafka |

## Install

```bash
# One skill at a time
npx skills add AdiPP/skills@skills/conventional-commits
npx skills add AdiPP/skills@skills/hyperf-backend
```

## Structure

Each skill follows the [Agent Skills spec](https://agentskills.io/specification.md):

```
skills/
├── conventional-commits/
│   └── SKILL.md
└── hyperf-backend/
    ├── SKILL.md
    └── references/
        ├── patterns.md
        ├── config-templates.md
        └── rules.md
```

## License

MIT
