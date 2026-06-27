# Agent Skills

Reusable AI coding agent skills in [Agent Skills](https://agentskills.io) format (YAML frontmatter + Markdown body).

## Available Skills

| Skill | Description |
|-------|-------------|
| [conventional-commits](skills/conventional-commits/) | Git commit message format following Conventional Commits spec |
| [hyperf-backend](skills/hyperf-backend/) | Hyperf microservice code generation with DDD, Clean Architecture, Kafka |
| [yaak](skills/yaak/) | Yaak API client CLI management (workspaces, folders, requests, envs) |

## Install

[skills.sh](https://skills.sh) compatible. Install via the [skills CLI](https://github.com/vercel-labs/skills):

```bash
# Install a specific skill
npx skills add AdiPP/skills@skills --skill conventional-commits
npx skills add AdiPP/skills@skills --skill hyperf-backend

# List available skills
npx skills add AdiPP/skills --list
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
