# Agent Skills

Monorepo of AI coding agent skills in [Agent Skills](https://agentskills.io) format (YAML frontmatter + Markdown body, same spec used by `skills.sh`). Distributed as individual GitHub repos; this repo is the development workspace.

## Repo Layout

```
skills/
  conventional-commits/    # Commit message style guide
    SKILL.md
  hyperf-backend/          # Hyperf microservice code generation
    SKILL.md
    references/
      patterns.md          # Full code templates
      config-templates.md  # Config file samples
      rules.md             # 31 enforced rules
docs/
  agentskills-io.md        # Spec requirements, frontmatter table, validation
  publishing-to-skills-sh.md  # Distribution steps
```

## Conventions (per SKILL.md)

- **SKILL.md body** MUST stay under 500 lines; heavy detail lives in `references/`.
- **Frontmatter** requires `name` (matches parent dir) and `description` (≤ 1024 chars, include trigger keywords).
- **One skill per published repo** — `SKILL.md` must sit at repo root for `npx skills add`.
- **Placeholder names**: `{ProjectName}` (PascalCase namespace), `{project-name}` (kebab-case Docker/Kafka), `{project_name}` (snake_case schema/env), `{Domain}` (bounded context), `{Action}` (verb in PascalCase), `{Suffix}` (role suffix for services).

## Commands

```bash
# Validate skill structure
skills-ref validate ./skills/<skill-name>

# Preview
cat skills/<name>/SKILL.md

# Search references
ls skills/<name>/references/
```

## Publishing

1. Publish each skill as its own public GitHub repo (skill root → repo root).
2. Install: `npx skills add <owner>/<repo>`.
3. `skills.sh` auto-indexes public repos with valid `SKILL.md`.

## What to Avoid

- Do not inline templates in `SKILL.md`; keep code samples in `references/`.
- Do not create multi-skill repos for distribution.
- Do not manually render diagrams here (this repo is docs/tooling only).
