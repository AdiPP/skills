# Agent Skills ecosystem

Sources: agentskills.io, skills.sh, spec at agentskills.io/specification.md

## What it is
A standardized SKILL.md format (YAML frontmatter + Markdown body) that many AI agents can load. Both `skills.sh` (Vercel) and `agentskills.io` refer to the same file format; `skills.sh` adds a distribution/leaderboard layer on top.

## Supported clients (agentskills.io home)
Claude Code, Claude, Gemini CLI, Cursor, Codex, Copilot, VS Code, OpenCode, OpenHands, Goose, Roo Code, Qodo, Laravel Boost, Spring AI, and ~20 others.

## Spec requirements (agentskills.io)

### Directory (one skill per dir)
```
skill-name/
  SKILL.md        # required
  scripts/        # optional
  references/     # optional
  assets/         # optional
```

### SKILL.md frontmatter
| Field | Required | Rule |
|---|---|---|
| name | yes | 1-64 chars, lowercase+numbers+hyphens, no leading/trailing/double hyphens, must match parent dir |
| description | yes | 1-1024 chars, what + when + specific trigger keywords |
| license | no | short name or bundled file reference |
| compatibility | no | up to 500 chars, env/product/tooling needs |
| metadata | no | arbitrary string→string map |
| allowed-tools | no | space-separated pre-approved tools (experimental) |

### Body
- No format restrictions.
- Recommended < 500 lines in SKILL.md.
- Heavy detail → `references/`, `scripts/`, `assets/`.
- File references: keep one level deep from SKILL.md root.

### Progressive disclosure
1. Metadata at startup (~100 tokens).
2. SKILL.md body when skill activates (< 5000 tokens recommended).
3. Reference files loaded on demand only.

## Validation
```bash
skills-ref validate ./skill-dir
```

## How our skills map to this
- Layout: `skills/<name>/SKILL.md` — matches spec.
- Name: `conventional-commits`, `hyperf-backend` — compliant.
- Description: trimmed to < 200 chars for skills.sh trigger; well within 1024 cap.
- hyperf-backend: 174 lines (refactored from 1563). Heavy content in `references/` for progressive disclosure.
- Files: `references/patterns.md`, `references/config-templates.md`, `references/rules.md`
