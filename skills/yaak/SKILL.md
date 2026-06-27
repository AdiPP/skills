---
name: yaak
description: Use when managing Yaak API client via CLI (`yaak` command). Covers workspace, folder, request, environment, cookie-jar, and auth management. Always read this skill before creating, updating, sending, or deleting Yaak resources from the terminal.
---

# Yaak CLI Skill

The `yaak` CLI (`@yaakapp/cli`) controls the Yaak desktop API client from the terminal. It reads/writes the same local SQLite database as the desktop app — changes are reflected instantly in both directions.

## Prerequisites

- `yaak` CLI installed: `npm install -g @yaakapp/cli`
- Yaak desktop app (optional but recommended for visual verification)

## Global flags

| Flag | Purpose |
|---|---|
| `--data-dir <DIR>` | Custom data directory |
| `-e, --environment <ID>` | Environment ID for variable substitution |
| `--cookie-jar <ID>` | Cookie jar ID for request execution |
| `-v, --verbose` | Verbose output (events, streamed body) |
| `--log [LEVEL]` | CLI logging (error\|warn\|info\|debug\|trace) |

## Command tree

```
yaak
├── workspace   list | show <id> | create | update <id> | delete <id> [--yes]
├── request     list [<ws-id>] | show <id> | create [<ws-id>] | update <id> | delete <id> [--yes] | send <id> | schema <type>
├── folder      list [<ws-id>] | show <id> | create -n <name> [<ws-id>] | update <id> | delete <id> [--yes]
├── environment list [<ws-id>] | show <id> | create | update <id> | delete <id> [--yes]
├── cookie-jar  list [<ws-id>]
├── send        <ID>                               # request, folder, or workspace
├── auth        login | logout | whoami
└── plugin      build | dev | generate | install | publish
```

**Multi-workspace rule:** When only one workspace exists, `[<ws-id>]` is optional. With multiple workspaces, you MUST pass the workspace ID as the first positional argument.

## Workspaces

```bash
yaak workspace list                                    # list all workspaces
yaak workspace show <ws-id>                            # full JSON
yaak workspace create -n "My API"                      # create by name
yaak workspace create --json '{"name":"My API","headers":[...]}'  # create with config
yaak workspace update <ws-id> --json '{"name":"Renamed"}'
yaak workspace delete <ws-id> --yes                    # non-interactive delete
```

## Folders

```bash
yaak folder list [<ws-id>]                             # list folders in workspace
yaak folder show <folder-id>                           # full JSON
yaak folder create -n "Auth" [<ws-id>]                 # create in workspace
yaak folder create --json '{"name":"Auth","workspaceId":"wk_..."}' [<ws-id>]
yaak folder update <folder-id> --json '{"name":"Renamed"}'
yaak folder delete <folder-id> --yes
```

## Requests

### List & inspect

```bash
yaak request list [<ws-id>]                            # list all requests
yaak request show <req-id>                             # full JSON (method, url, headers, body, etc.)
```

### Create

Quick flags:
```bash
yaak request create [<ws-id>] -n "Get Users" -m GET -u "https://api.example.com/users"
```

Full JSON:
```bash
yaak request create [<ws-id>] --json '{
  "name": "Create User",
  "method": "POST",
  "url": "https://api.example.com/users",
  "folderId": "fl_...",
  "bodyType": "application/json",
  "body": {
    "text": "{\"name\":\"test\",\"email\":\"test@example.com\"}"
  },
  "headers": [
    {"name":"Authorization","value":"Bearer token","enabled":true}
  ]
}'
```

**Key fields for `--json`:**

| Field | Type | Notes |
|---|---|---|
| `name` | string | Display name |
| `method` | string | GET, POST, PUT, PATCH, DELETE |
| `url` | string | Request URL |
| `folderId` | string | Parent folder ID |
| `bodyType` | string | `application/json`, `multipart/form-data`, `application/x-www-form-urlencoded`, `text/plain`, etc. |
| `body.text` | string | Body content (stringify JSON for application/json) |
| `headers` | array | `[{name, value, enabled}]` |
| `authenticationType` | string | `basic`, `bearer`, `oauth2`, etc. |
| `authentication` | object | Auth config per type |

### Update

```bash
yaak request update <req-id> --json '{"name":"Updated Name","url":"https://new-url.com"}'
```

### Delete

```bash
yaak request delete <req-id> --yes
```

### Send

```bash
yaak send <req-id>                                     # send a single request
yaak send <req-id> -v                                  # verbose (headers, events)
yaak send <folder-id>                                  # send all requests in folder
yaak send <ws-id> --parallel --fail-fast               # send entire workspace
```

**Send flags:**
- `--parallel` — execute requests concurrently
- `--fail-fast` — abort on first failure (works with folder/workspace sends)
- `-e <env-id>` — use environment for variable substitution
- `-v` — verbose output (shows request/response headers, streamed body)

### Schema introspection

```bash
yaak request schema http       # JSON Schema for HTTP requests
yaak request schema grpc       # JSON Schema for gRPC requests
yaak request schema websocket  # JSON Schema for WebSocket requests
```

Use this to understand the exact `--json` payload structure for create/update.

## Environments

```bash
yaak environment list [<ws-id>]                        # list environments
yaak environment show <env-id>                         # full JSON with variables
yaak environment create [<ws-id>] -n "Production"
yaak environment update <env-id> --json '{
  "name": "Production",
  "variables": [
    {"name":"BASE_URL","value":"https://api.prod.com"},
    {"name":"API_KEY","value":"secret123"}
  ]
}'
yaak environment delete <env-id> --yes
```

### Using environments

Pass `-e <env-id>` to any `send` command for variable substitution:

```bash
yaak send <req-id> -e <env-id>
```

**Template syntax in requests:**
- Variables: `${[ my_var ]}`
- Functions: `${[ uuid() ]}`, `${[ timestamp() ]}`, `${[ fs.readFile('/path') ]}`

## Auth

```bash
yaak auth whoami                                       # check login status
yaak auth login                                        # login via browser
yaak auth logout                                       # sign out
```

## Batch operations

### Create multiple requests from a script

```bash
#!/bin/bash
WS="wk_..."
FL_AUTH="fl_..."
FL_PROD="fl_..."

# Auth endpoints
yaak request create "$WS" --json '{"name":"Login","method":"POST","url":"https://api.example.com/auth/login","folderId":"'"$FL_AUTH"'","bodyType":"application/json","body":{"text":"{\"username\":\"user\",\"password\":\"pass\"}"}}'
yaak request create "$WS" --json '{"name":"Me","method":"GET","url":"https://api.example.com/auth/me","folderId":"'"$FL_AUTH"'"}'

# CRUD endpoints
for op in "GET list /users" "POST create /users" "GET get /users/1" "PUT update /users/1" "DELETE delete /users/1"; do
  read -r method label path <<< "$op"
  yaak request create "$WS" --json '{"name":"'"${label^}"' '"${path#/}"'","method":"'"$method"'","url":"https://api.example.com'"$path"'","folderId":"'"$FL_PROD"'"}'
done
```

### Delete all requests in a workspace

```bash
yaak request list <ws-id> | awk '{print $1}' | while read id; do
  yaak request delete "$id" --yes
done
```

### Clone workspace structure to new workspace

```bash
# Create new workspace
NEW_WS=$(yaak workspace create -n "Copy" | awk '{print $NF}')

# Create folders
yaak folder list <old-ws> | while read id name; do
  yaak folder create -n "$name" "$NEW_WS"
done

# Then create requests with new folder IDs
```

## JSON output

All `show` and `list` commands output JSON (or one-line-per-item for `list`). Pipe to `jq` for filtering:

```bash
yaak request show <req-id> | jq '.url'
yaak workspace list | jq -r '.[] | .id + " " + .name'
```

## Common patterns

### Organize by resource

Create a folder per API resource, then CRUD requests inside:

```bash
WS="wk_..."
FL=$(yaak folder create -n "Users" "$WS" | awk '{print $NF}')

yaak request create "$WS" --json '{"name":"List Users","method":"GET","url":"https://api.example.com/users","folderId":"'"$FL"'"}'
yaak request create "$WS" --json '{"name":"Get User","method":"GET","url":"https://api.example.com/users/1","folderId":"'"$FL"'"}'
yaak request create "$WS" --json '{"name":"Create User","method":"POST","url":"https://api.example.com/users","folderId":"'"$FL"'","bodyType":"application/json","body":{"text":"{\"name\":\"new user\"}"}}'
```

### Environment-based URL switching

```bash
# Dev environment
yaak environment create "$WS" -n "Dev"
yaak environment update <dev-env-id> --json '{"variables":[{"name":"BASE_URL","value":"http://localhost:3000"}]}'

# Prod environment
yaak environment create "$WS" -n "Prod"
yaak environment update <prod-env-id> --json '{"variables":[{"name":"BASE_URL","value":"https://api.production.com"}]}'

# Use in request URL
yaak request create "$WS" --json '{"name":"Get Users","method":"GET","url":"${[ BASE_URL ]}/users","folderId":"..."}'

# Send with env
yaak send <req-id> -e <dev-env-id>    # hits localhost
yaak send <req-id> -e <prod-env-id>   # hits production
```

### Workspace-level defaults

Set headers/auth at workspace level (inherited by all requests):

```bash
yaak workspace update "$WS" --json '{
  "headers": [
    {"name":"Content-Type","value":"application/json","enabled":true},
    {"name":"Accept","value":"application/json","enabled":true}
  ],
  "authenticationType": "bearer",
  "authentication": {"token":"${[ API_TOKEN ]}"}
}'
```

## Tips

- Always `yaak request schema <type>` before creating complex payloads — the schema shows exact field names and types.
- Use `--verbose` on `send` to debug request/response details.
- The CLI is idempotent for reads — safe to script and re-run.
- Desktop app reflects CLI changes instantly (shared SQLite DB).
- `--yes` skips confirmation prompts — essential for scripted deletes.
