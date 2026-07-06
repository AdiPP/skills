---
name: atlassian-cli
description: "Use when managing Atlassian Cloud products from the terminal. Covers Jira, Confluence, Bitbucket, JSM, Opsgenie, Bamboo, and authentication. Trigger keywords: atlassian-cli, jira cli, confluence cli, bitbucket cli, jsm cli, atlassian cloud cli, jira issue, confluence page, bitbucket pipeline."
---

# Atlassian CLI Skill

`atlassian-cli` v0.4.2 by omar16100: unified Rust CLI for Atlassian Cloud products. Credentials stored encrypted via OS keychain. Installed via Homebrew.

## Prerequisites

```bash
brew install atlassian-cli/tap/atlassian-cli
```

Source: https://github.com/omar16100/atlassian-cli

## Global flags

| Flag | Purpose |
|---|---|
| `-p, --profile <PROFILE>` | Profile to use from config file |
| `--config <PATH>` | Path to config file (defaults to `~/.atlassian-cli/config.yaml`) |
| `-f, --format <FORMAT>` | Output format: table (default), json, yaml, csv, quiet, markdown |
| `--envelope` | Wrap JSON/YAML list output in `{"data": [...], "count": N}` envelope |
| `--debug` | Enable verbose logging |

## Auth

Credentials per-profile. `--profile` switches between sites.

```bash
# Jira/Confluence: API token + base URL
atlassian-cli auth login --profile work --base-url https://example.atlassian.net --email user@example.com

# Bitbucket: API token (Basic auth) or access token (Bearer auth)
atlassian-cli auth login --profile work --bitbucket --email user@example.com
atlassian-cli auth login --profile work --bitbucket --bearer

# Use ATLASSIAN_API_TOKEN env var (skips prompt)
export ATLASSIAN_API_TOKEN=your_token_here

# Mark as default, test, status
atlassian-cli auth login --profile work --base-url https://example.atlassian.net --default
atlassian-cli auth test
atlassian-cli auth test --bitbucket
atlassian-cli auth status
atlassian-cli auth whoami
atlassian-cli auth list

# Logout
atlassian-cli auth logout --profile work
atlassian-cli auth logout --profile work --remove-profile
atlassian-cli auth logout --profile work --bitbucket
```

**Tokens:** Create at https://id.atlassian.com/manage-profile/security/api-tokens. App passwords deprecated — use API tokens or Bearer.

## Jira

```
atlassian-cli jira
├── issue       search | get | create | update | delete | transition | assign | unassign
│               watchers (list|add|remove) | links (list|create|delete) | comments (list|add|update|delete)
├── project     list | get | create | update | delete
├── components  list | get | create | update | delete
├── versions    list | get | create | update | delete | merge
├── roles       list | get | actors | add-actor | remove-actor
├── fields      list | get | create | delete
├── workflows   list | get | export
├── bulk        transition | assign | label | export | import
├── automation  list | get | create | update | enable | disable | delete | export
├── webhooks    list | get | create | update | enable | disable | delete | test
└── audit       list | export
```

### Issue operations

```bash
# Search
atlassian-cli jira issue search --project PROJ --status "In Progress" --assignee @me
atlassian-cli jira issue search --type Bug --priority High --limit 50
atlassian-cli jira issue search --jql 'project = PROJ AND status = Open ORDER BY created DESC'
atlassian-cli jira issue search --text "login error" --label frontend --show-query

# CRUD
atlassian-cli jira issue get PROJ-123

atlassian-cli jira issue create --project PROJ --issue-type Bug --summary "Fix login error"
atlassian-cli jira issue create --project PROJ --issue-type Story --summary "Add feature" \
  --description "Detailed description" --assignee user@email.com --priority High \
  --sprint 42 --field 'customfield_10001={"value":"Alpha"}'

atlassian-cli jira issue update PROJ-123 --summary "New title" --priority Medium \
  --field 'customfield_10020={"formula":"a=b"}'

atlassian-cli jira issue delete PROJ-123 --force
atlassian-cli jira issue transition PROJ-123 --transition "In Progress"

# Assignment
atlassian-cli jira issue assign PROJ-123 --assignee user@email.com
atlassian-cli jira issue unassign PROJ-123

# Comments, watchers, links
atlassian-cli jira issue comments list PROJ-123
atlassian-cli jira issue comments add PROJ-123 --body "Looking into this"
atlassian-cli jira issue watchers add PROJ-123 user@email.com
atlassian-cli jira issue links create PROJ-123 PROJ-456 --link-type blocks
```

**Description:** Markdown auto-converted to ADF (headings, lists, bold, italic, inline code, links, code blocks).

**Custom fields:** Discover IDs with `jira fields list`. `--field key=json_value` (repeatable). Cannot collide with typed flags.

### Project, components, versions, roles, fields

```bash
atlassian-cli jira project list
atlassian-cli jira project get PROJ
atlassian-cli jira components list --project PROJ
atlassian-cli jira versions list --project PROJ
atlassian-cli jira versions merge --project PROJ --from VER-1 --to VER-2
atlassian-cli jira roles list --project PROJ
atlassian-cli jira roles actors --project PROJ --role-id 10001
atlassian-cli jira roles add-actor --project PROJ --role-id 10001 --user user@email.com
atlassian-cli jira fields list
atlassian-cli jira fields get customfield_10001
```

### Workflows

```bash
atlassian-cli jira workflows list
atlassian-cli jira workflows get <workflow-id>
atlassian-cli jira workflows export <workflow-id>
```

### Bulk operations

```bash
atlassian-cli jira bulk transition --jql 'project = PROJ AND status = Open' --transition "In Progress" --dry-run
atlassian-cli jira bulk assign --jql 'project = PROJ' --assignee user@email.com
atlassian-cli jira bulk label --jql 'project = PROJ' --action set --labels triaged
atlassian-cli jira bulk export --jql 'project = PROJ' --output issues.json
atlassian-cli jira bulk import --file issues.json --project PROJ --dry-run
```

Bulk ops support `--dry-run`, `--concurrency N` (default 4).

### Automation & webhooks

```bash
atlassian-cli jira automation list
atlassian-cli jira automation enable <rule-id>
atlassian-cli jira webhooks create --name "My Webhook" --url https://example.com/webhook --events issue_created
atlassian-cli jira webhooks test <webhook-id>
```

### Audit

```bash
atlassian-cli jira audit list --limit 50
atlassian-cli jira audit export --output audit.json
```

## Confluence

```
atlassian-cli confluence
├── space       list | get | create | update | delete | permissions | add-permission
├── page        list | get | create | update | publish | delete | versions | add-label | remove-label
│               comments | add-comment | get-restrictions | add-restriction | remove-restriction
├── folder      get | create | delete
├── blog        list | get | create | update | publish | delete
├── attachment  list | get | upload | download | delete
├── search      cql | text | in-space | params
├── bulk        delete | add-labels | export
└── analytics   page-views | space-stats
```

```bash
# Spaces
atlassian-cli confluence space list
atlassian-cli confluence space get KEY
atlassian-cli confluence space create --key DOCS --name "Documentation" --description "Project docs"

# Pages
atlassian-cli confluence page list --space KEY
atlassian-cli confluence page create --space KEY --title "Getting Started" --body content.html
atlassian-cli confluence page update <page-id> --title "New Title" --body content.html
atlassian-cli confluence page publish <page-id> --body content.html --message "First publish"

# Labels & comments
atlassian-cli confluence page add-label <page-id> --label howto
atlassian-cli confluence page comments <page-id>

# Restrictions
atlassian-cli confluence page get-restrictions <page-id>
atlassian-cli confluence page add-restriction <page-id> --user user@email.com --restriction view

# Attachments
atlassian-cli confluence attachment list <page-id>
atlassian-cli confluence attachment upload <page-id> file.pdf
atlassian-cli confluence attachment download <attachment-id> output.pdf

# Folders (v2 API)
atlassian-cli confluence folder create --space KEY --name "Reports"
atlassian-cli confluence folder get <folder-id>

# Blog
atlassian-cli confluence blog list --space KEY
atlassian-cli confluence blog create --space KEY --title "Release Notes" --body content.html
atlassian-cli confluence blog publish <blog-id> --body content.html

# Search
atlassian-cli confluence search cql --cql 'type=page and space=DOCS'
atlassian-cli confluence search text --text "API reference" --limit 20

# Bulk & analytics
atlassian-cli confluence bulk delete --cql 'type=page and space=TMP' --dry-run
atlassian-cli confluence bulk export --cql 'space=DOCS' --output pages.json
atlassian-cli confluence analytics page-views <page-id>
atlassian-cli confluence analytics space-stats KEY
```

Page body is HTML storage format from file, not inline. Use `attachment upload|download` with file paths.

## Bitbucket

Alias: `bb`.

```
atlassian-cli bb
├── repo        list | get | create | update | delete
├── branch      list | get | create | delete | protect | unprotect | restrictions
├── pr          list | get | create | update | merge | decline | approve | unapprove | diff
│               comments | comment | reviewers
├── workspace   list | get
├── project     list | get | create | update | delete
├── pipeline    list | get | latest | trigger | stop | logs | watch | steps | status | rerun | var | env
├── webhook     list | create | delete
├── ssh-key     list | add | delete
├── permission  list | grant | revoke
├── commit      list | get | diff | browse
├── bulk        archive-repos | delete-branches
└── whoami
```

`--workspace` defaults to the profile base URL host prefix. `--repo` auto-detected from git remote when in a repo dir.

```bash
# Repos & branches
atlassian-cli bb repo list
atlassian-cli bb repo create --name my-repo --project PROJ --private --fork-policy no_forks
atlassian-cli bb branch list
atlassian-cli bb branch create --branch feature/new-api --from main
atlassian-cli bb branch protect --branch main --types push --types delete
atlassian-cli bb branch restrictions

# Pull requests
atlassian-cli bb pr list --state OPEN
atlassian-cli bb pr create --title "Add feature" --source feature/new-api --target main
atlassian-cli bb pr merge <pr-id> --merge-strategy squash
atlassian-cli bb pr approve <pr-id>
atlassian-cli bb pr diff <pr-id>
atlassian-cli bb pr reviewers <pr-id> --add user@email.com

# Pipelines
atlassian-cli bb pipeline list
atlassian-cli bb pipeline trigger --branch main
atlassian-cli bb pipeline watch <pipeline-id>
atlassian-cli bb pipeline status
atlassian-cli bb pipeline rerun <pipeline-id>
atlassian-cli bb pipeline var --key DEPLOY_KEY --value "secret" --secured
atlassian-cli bb pipeline env list

# SSH keys, permissions, commits
atlassian-cli bb ssh-key add --key ~/.ssh/id_ed25519.pub --label "My key"
atlassian-cli bb permission grant --user user@email.com --permission write
atlassian-cli bb commit list --branch main --limit 10
atlassian-cli bb commit get <commit-hash>
atlassian-cli bb commit diff <commit-hash>
atlassian-cli bb commit browse <commit-hash>

# Bulk
atlassian-cli bb bulk archive-repos --days 365 --dry-run
atlassian-cli bb bulk delete-branches --merged-before "2025-01-01"
```

## JSM (Jira Service Management)

```
atlassian-cli jsm
├── service-desk  list | get | customers | add-customer | remove-customer | organizations | add-organization | remove-organization
├── request       list | get | create | transitions | transition | status | comments | add-comment
│                 participants | add-participant | remove-participant | subscribe | unsubscribe
├── queue         list | get | issues
├── approval      list | get | approve | decline
├── sla           list | get
├── customer      create | revoke-portal-access
├── organization  list | get | create | delete | users | add-user | remove-user
├── request-type  list | get | fields | groups
├── kb            search
└── feedback      get | submit | delete
```

```bash
# Service desks & requests
atlassian-cli jsm service-desk list
atlassian-cli jsm service-desk get <desk-id>
atlassian-cli jsm request list --service-desk <desk-id>
atlassian-cli jsm request create --service-desk <desk-id> --request-type-id <type-id> \
  --summary "Need VPN access" --description "Cannot connect from home"

# Queues, approvals, SLAs
atlassian-cli jsm queue issues <desk-id> <queue-id>
atlassian-cli jsm approval approve <request-id> --comment "Looks good"
atlassian-cli jsm sla list <request-id>

# Customers & organizations
atlassian-cli jsm customer create --email user@example.com --display-name "Jane Doe"
atlassian-cli jsm organization create --name "Acme Corp"

# KB & feedback
atlassian-cli jsm kb search --query "VPN setup" --service-desk <desk-id>
atlassian-cli jsm feedback submit <request-id> --rating 5 --comment "Great support"
```

## Opsgenie

```
atlassian-cli opsgenie
├── alert       list | get | create | close | ack | unack | snooze | escalate | assign | add-note | delete | recipients | logs | notes
├── incident    list | get | create | close | resolve | reopen | add-responder | add-note | delete | timeline
├── schedule    list | get | create | delete | enable | disable | on-call | timeline | export
├── team        list | get | create | delete | members | add-member | remove-member | on-call
├── escalation  list | get | create | delete
├── service     list | get | create | delete
└── heartbeat   list | get | create | delete | enable | disable | ping
```

```bash
# Alerts
atlassian-cli opsgenie alert list --status open --priority P2
atlassian-cli opsgenie alert create --message "CPU > 90%" --priority P1 --team team-id
atlassian-cli opsgenie alert ack <alert-id>
atlassian-cli opsgenie alert close <alert-id> --note "Resolved by automation"

# Incidents
atlassian-cli opsgenie incident create --message "Production outage" --priority P1
atlassian-cli opsgenie incident resolve <incident-id>

# Schedules & teams
atlassian-cli opsgenie schedule on-call <schedule-id> --date "2025-01-15"
atlassian-cli opsgenie schedule export <schedule-id> --output calendar.ics
atlassian-cli opsgenie team on-call <team-id>

# Heartbeats
atlassian-cli opsgenie heartbeat ping <heartbeat-name>
```

## Bamboo

```
atlassian-cli bamboo
├── project   list | get
├── plan      list | get | enable | disable | favorite | unfavorite
├── branch    list | get | create | delete | enable | disable
├── build     list | get | latest | run | stop | logs | comment | add-label | remove-label
├── deploy    projects | project | environments | environment | results | trigger | versions
├── agent     list | get | enable | disable | capabilities
├── artifact  list | download
├── info      (server information)
└── queue     (build queue status)
```

```bash
# Plans & branches
atlassian-cli bamboo plan list --project PROJ
atlassian-cli bamboo plan enable <plan-key>
atlassian-cli bamboo branch create <plan-key> --branch feature/new

# Builds
atlassian-cli bamboo build run <plan-key> --branch main
atlassian-cli bamboo build latest <plan-key> --branch main
atlassian-cli bamboo build logs <build-id>

# Deploy & artifacts
atlassian-cli bamboo deploy trigger <deploy-project-id> --environment production --version 1.2.3
atlassian-cli bamboo artifact download <build-id> --artifact-name "my-app.jar"

# Agents
atlassian-cli bamboo agent list
atlassian-cli bamboo agent capabilities <agent-id>
```

## Output formats

All commands accept `-f json | yaml | csv | quiet | markdown`.

```bash
# JSON for scripting
atlassian-cli jira issue search --project PROJ -f json | jq '.[] | .key + " " + .fields.summary'

# Quiet mode — exit code only
atlassian-cli jira issue get PROJ-123 -f quiet && echo "Issue exists" || echo "Not found"

# Envelope wraps JSON/YAML list output
atlassian-cli jira issue search --project PROJ -f json --envelope
```

## Common patterns

### Search and bulk-update

```bash
atlassian-cli jira bulk transition --jql 'project = PROJ AND type = Bug AND status = Open' \
  --transition "In Progress" --dry-run
```

### Export and re-import

```bash
atlassian-cli jira bulk export --jql 'project = PROJ AND updated >= -7d' --output weekly.json
atlassian-cli jira bulk import --file weekly.json --project PROJ --dry-run
```

### Pipeline watch and status

```bash
atlassian-cli bb pipeline trigger --branch main
atlassian-cli bb pipeline watch <uuid>
if atlassian-cli bb pipeline status --branch main -f quiet; then
  echo "Pipeline passed"
else
  echo "Pipeline failed"
fi
```

### Multi-profile

```bash
atlassian-cli -p work jira issue get PROJ-123
atlassian-cli -p personal jira issue get SCRUM-42
ATLASSIANS_API_TOKEN=$(pass atlassian/token) atlassian-cli auth login --profile ci
```

### Bitbucket auto-detection

When run inside a git repo matching a configured Bitbucket workspace, `--repo` is auto-detected.

```bash
cd my-repo
atlassian-cli bb pr list
atlassian-cli -p work bb pr create --title "Fix" --source feat/x --target main
```
