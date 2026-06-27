---
name: conventional-commits
description: Write git commit messages following Conventional Commits spec. Covers format, types, scope, ticket codes from branch names, body, footer, and breaking changes. Use when composing commit messages.
---

# Conventional Commit Messages

Opinionated conventional commit format. Spec-faithful; tightens a few style rules.

## Format

```
<type>(<optional scope>): <description>

<optional body>

<optional footer>
```

- **Header line ≤ 72 chars** (subject + blank line is the visual unit).
- Blank line between header / body / footer (mandatory when any are present).
- Subject is the only required part.

## Types

| Type | Use for |
|---|---|
| `feat` | Add, adjust, or remove a user-facing/API feature |
| `fix` | Fix a `feat` bug |
| `refactor` | Restructure code without changing behavior |
| `perf` | `refactor` that specifically improves performance |
| `style` | Whitespace, formatting, semicolons — no behavior change |
| `test` | Add or correct tests |
| `docs` | Documentation only |
| `build` | Build tools, dependencies, project version |
| `ops` | Infra, deployment, CI/CD, monitoring, backups |
| `chore` | Initial commit, `.gitignore`, misc tasks |

## Scope

- Optional. Project-defined (e.g. `api`, `cli`, `auth`, `shopping-cart`).
- **Do not** use issue IDs as scopes — those belong in the footer.
- Keep short (≤ 20 chars is the gist's server-side rule of thumb).

## Description (subject)

- **Imperative, present tense**: "change", not "changed"/"changes". Frame as "This commit will…".
- **Lowercase first letter** (no capital after the type/scope colon).
- **No trailing period**.
- Concise — one line, no body needed for trivial changes.

## Ticket code

If the current branch name contains a ticket code (e.g., `SDIT-1222`), include it in the subject line after the colon:

```
<type>(<optional scope>): [<ticket>] <description>
```

**How to extract:**
1. Run `git branch --show-current` to get the branch name
2. Match pattern `[A-Z]+-\d+` (e.g., `SDIT-1222`, `FE-456`)
3. If found, prepend `[TICKET]` to the description

**Examples:**
```
feat: [SDIT-1222] add category field to event model
fix: [SDIT-1296] add suffix in webhook tracing client name
refactor: [SDIT-1215] create empty webhook client with provided id and secret
```

If no ticket code in branch name, skip this step.

## Body (optional)

- Wrap at ~72 cols.
- Imperative, present tense.
- Explain **why**, contrast with previous behavior. Omit for trivial changes.

## Footer (optional)

- Reference issues: `Closes #123`, `Fixes JIRA-456`.
- **Breaking changes must** start with `BREAKING CHANGE:` (single line: space after colon; multi-line: two newlines after colon).
- See BREAKING CHANGES below.

## Breaking changes

- Append `!` **before** the colon in the subject: `feat(api)!: remove status endpoint`.
- **Must** also describe in the footer with `BREAKING CHANGE:` — both, not either-or.
- Bumps **major** version.

## Versioning hint

- Breaking change → **major** bump.
- `feat` or `fix` → **minor** bump.
- Anything else (`refactor`, `chore`, `docs`, …) → **patch** bump.

## Examples

```
feat: add email notifications on new direct messages
```

```
feat(shopping cart): add the amazing button
```

```
fix(shopping-cart): prevent order an empty shopping cart
```

```
fix(api): fix wrong calculation of request body checksum
```

```
fix: add missing parameter to service call

The error occurred due to <reasons>.
```

```
perf: decrease memory footprint for unique visitors by using HyperLogLog
```

```
build: update dependencies
```

1. Pick the right type from the table (most common mistake: `feat` vs `fix` vs `refactor`).
2. Pick a scope only if the project uses them; otherwise omit.
3. Extract ticket code from branch name if present (see Ticket code section).
4. Subject ≤ 72 chars, imperative, lowercase, no period.
5. Body only if the why isn't obvious.
6. Footer: issue refs and/or `BREAKING CHANGE:` if applicable.
7. Verify with the regex (the gist's server-side pre-receive rule):
   `^(feat|fix|refactor|style|test|docs|build|ops|chore|perf)(\(.{1,20}\))?!?: .{1,100}$`
