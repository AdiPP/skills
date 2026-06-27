# How to publish skills to skills.sh

Source: skills.sh, vercel-labs/skills CLI.

There is no "submit" button. skills.sh is a community-driven directory backed by GitHub repos.

## Distribution model

```
GitHub public repo  →  npx skills add <owner>/<repo>  →  agent loads SKILL.md
                       ↓
                  skills.sh scrapes GitHub for leaderboard / install counts
```

One skill per repo. The CLI expects the repo root to contain `SKILL.md`.

## Steps

1. **Create a skill directory** with at minimum a `SKILL.md` file:
   ```
   skill-name/
     SKILL.md        # required: YAML frontmatter + markdown body
     references/     # optional: heavy details for progressive disclosure
     scripts/        # optional: executable code
     assets/         # optional: templates, resources
   ```

2. **Push to a public GitHub repo**:
   ```bash
   git init
   git add .
   git commit -m "chore: initial skill"
   git remote add origin https://github.com/<owner>/<repo>.git
   git push origin main
   ```

3. **Install immediately**:
   ```bash
   npx skills add <owner>/<repo>
   ```

4. **Get indexed on skills.sh** — check `vercel-labs/skills` for contribution guidelines (likely an issue or PR to register the skill in a registry file). Even without that, `npx skills find` still surfaces public repos with valid `SKILL.md`.

## Key rules

- **One skill per repo**. One repo with multiple `SKILL.md` files = not installable with `npx skills add` without extra config.
- **SKILL.md at repo root** — the CLI does not look in subdirectories.
- **Frontmatter required** with at least `name` and `description` (see `docs/agentskills-io.md` for full spec).

## Our skills layout

We keep multiple skills in this repo for development (`skills/conventional-commits/`, `skills/hyperf-backend/`). To publish, each skill must live in its own GitHub repo:

```
your-username/conventional-commits   →  skills/conventional-commits/SKILL.md
your-username/hyperf-backend         →  skills/hyperf-backend/SKILL.md
```

Monorepo is fine for dev; distribution is per-repo.
