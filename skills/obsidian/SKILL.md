---
name: obsidian
description: Manage Obsidian vaults via CLI (`obsidian` command). Covers notes, daily notes, search, tags, properties, tasks, templates, backlinks, plugins, themes, sync, and developer tools. Use when reading, creating, editing, searching, or organizing notes in Obsidian from the terminal.
---

# Obsidian CLI Skill

The `obsidian` CLI controls the Obsidian desktop app from the terminal. It communicates with the running app — Obsidian must be open.

## Prerequisites

- Obsidian desktop app running
- `obsidian` CLI installed and on PATH

## How file arguments work

| Form | Behavior |
|---|---|
| `file=<name>` | Wikilink-style name matching (fuzzy, resolves by note name) |
| `path=<path>` | Exact filesystem path (`folder/note.md`) |
| Omitted | Defaults to the active file in Obsidian |

Quote values with spaces: `name="My Note"`. Use `\n` for newline, `\t` for tab in content values.

## Vault targeting

Pass `vault=<name>` to target a specific vault. Omit when only one vault exists.

## Command tree

```
obsidian
├── File ops:      create | read | append | prepend | delete | move | rename | open | file | files | folders | folder
├── Daily notes:   daily | daily:read | daily:append | daily:prepend | daily:path
├── Search:        search | search:context | search:open
├── Tags:          tags | tag
├── Properties:    properties | property:read | property:set | property:remove
├── Tasks:         tasks | task
├── Templates:     templates | template:read | template:insert
├── Links:         backlinks | links | orphans | deadends | unresolved
├── History:       history | history:list | history:read | history:restore | history:open
├── Sync:          sync | sync:status | sync:history | sync:read | sync:restore | sync:deleted | sync:open
├── Plugins:       plugins | plugins:enabled | plugins:restrict | plugin | plugin:install | plugin:enable | plugin:disable | plugin:reload | plugin:uninstall
├── Themes:        themes | theme | theme:install | theme:set | theme:uninstall
├── Bookmarks:     bookmarks | bookmark
├── Tabs:          tabs | tab:open
├── Bases:         bases | base:create | base:query | base:views
├── Snippets:      snippets | snippets:enabled | snippet:enable | snippet:disable
├── Dev tools:     eval | devtools | dev:dom | dev:css | dev:console | dev:errors | dev:screenshot | dev:debug | dev:cdp | dev:mobile
└── Misc:          vault | vaults | reload | restart | version | random | random:read | recents | outline | aliases | wordcount | commands | command | hotkeys | hotkey | workspace | diff
```

## File operations

### create

```bash
obsidian create name="My Note" content="# Hello"
obsidian create path=journal/entry.md template=daily overwrite open
obsidian create name="Project" content="# Notes" newtab
```

| Param | Purpose |
|---|---|
| `name=<name>` | File name (wikilink resolution) |
| `path=<path>` | Exact file path |
| `content=<text>` | Initial content |
| `template=<name>` | Template to use |
| `overwrite` | Overwrite if exists |
| `open` | Open after creating |
| `newtab` | Open in new tab |

### read / append / prepend

```bash
obsidian read file="Meeting Notes"                          # read by name
obsidian read path=journal/2024-01-01.md                    # read by path
obsidian append path=log.md content="New entry\nline two"   # append content
obsidian prepend path=log.md content="# Header"             # prepend content
obsidian append path=log.md content="inline text" inline    # no trailing newline
```

`append` and `prepend` require `content=<text>`.

### delete / move / rename / open

```bash
obsidian delete file="Draft"                                # trash (safe)
obsidian delete path=temp.md permanent                       # skip trash
obsidian move file=Chapter1 to="Archive/Chapter1"            # move to folder
obsidian rename file=WIP name="Final"                        # rename file
obsidian open path=dashboard.md                              # open in current tab
obsidian open file="Daily" newtab                            # open in new tab
```

### file / files / folders / folder — list and inspect

```bash
obsidian file path=note.md                                  # file info (metadata)
obsidian files                                              # all files
obsidian files folder=Projects ext=.md total                # filtered + count
obsidian folders                                            # all folders
obsidian folders folder=Projects total                      # subfolder count
obsidian folder path=References                             # folder info
obsidian folder path=References info=size                   # size only
```

`files` supports `folder=<path>`, `ext=<extension>`, `total`. `folder` supports `info=files|folders|size`.

## Daily notes

```bash
obsidian daily                                              # open today's daily
obsidian daily paneType=split                               # open in split pane
obsidian daily:read                                         # read today's daily
obsidian daily:append content="Task: review PR"             # append to daily
obsidian daily:prepend content="## Morning"                 # prepend to daily
obsidian daily:append content="line" inline                 # append without newline
obsidian daily:path                                         # get daily note path
```

Use `open` flag on append/prepend to open the file after adding content. `paneType=tab|split|window` controls where daily opens.

## Search

```bash
obsidian search query="meeting notes"                       # search vault
obsidian search query="API key" path=Projects limit=10      # filtered + limited
obsidian search query="TODO" case                           # case-sensitive
obsidian search query="schema" total                        # match count only
obsidian search:context query="function" format=json        # with line context
obsidian search:open query="@mentioned"                     # open search view
```

| Param | Purpose |
|---|---|
| `query=<text>` | Search query (required) |
| `path=<folder>` | Limit to folder |
| `limit=<n>` | Max results |
| `case` | Case-sensitive |
| `total` | Match count only |
| `format=text|json` | Output format |

## Tags

```bash
obsidian tags                                               # all vault tags
obsidian tags counts                                        # with occurrence counts
obsidian tags sort=count format=json                        # sorted by count
obsidian tags file="Note"                                   # tags on a file
obsidian tags active                                        # tags on active file
obsidian tag name=project                                   # tag info + files
obsidian tag name=project verbose                           # include file list
```

## Properties

```bash
obsidian properties                                         # all vault properties
obsidian properties file="Note"                             # properties on a file
obsidian properties name=status counts                      # property occurrence count
obsidian property:read name=status file="Note"              # read property value
obsidian property:set name=status value=done type=text      # set property
obsidian property:set name=priority value=high type=list    # list property type
obsidian property:remove name=status file="Note"            # remove property
```

`property:set` supports `type=text|list|number|checkbox|date|datetime`.

## Tasks

```bash
obsidian tasks                                              # all incomplete tasks
obsidian tasks done                                         # completed tasks
obsidian tasks todo                                         # incomplete tasks (alt)
obsidian tasks status="?"                                   # tasks with specific status
obsidian tasks file="Project" verbose                       # tasks grouped by file
obsidian tasks active                                       # active file tasks
obsidian tasks daily                                        # daily note tasks
obsidian task ref="note.md:42" toggle                       # toggle task status
obsidian task path=note.md line=42 done                     # mark task done
obsidian task path=note.md line=42 status="?"               # set custom status
```

`task` targets a specific task by `ref=<path:line>` or `file`+`line`. Supported actions: `toggle`, `done`, `todo`, `status="<char>"`.

## Templates

```bash
obsidian templates                                          # list templates
obsidian template:read name=standard                        # read template content
obsidian template:read name=project resolve                 # resolve variables
obsidian template:insert name=standard                      # insert into active file
```

`template:read` with `resolve` substitutes template variables. Optionally pass `title=<title>` for variable resolution context.

## Links and backlinks

```bash
obsidian backlinks path=note.md                             # incoming links
obsidian backlinks file="Note" counts total                 # link counts
obsidian backlinks path=note.md format=json                 # JSON output
obsidian links file="Note"                                  # outgoing links
obsidian links path=note.md total                           # outgoing link count
obsidian orphans                                            # no incoming links
obsidian deadends                                           # no outgoing links
obsidian unresolved                                         # broken wikilinks
```

`orphans` and `deadends` support `all` (include non-markdown) and `total`. `unresolved` supports `counts`, `verbose`, `total`, `format`.

## History

```bash
obsidian history path=note.md                               # version history
obsidian history:list                                       # files with history
obsidian history:read path=note.md version=3                # read old version
obsidian history:restore path=note.md version=2             # restore version
obsidian history:open path=note.md                          # open recovery view
```

## Sync

```bash
obsidian sync on                                            # resume sync
obsidian sync off                                           # pause sync
obsidian sync:status                                        # sync status
obsidian sync:history path=note.md                          # sync versions
obsidian sync:read path=note.md version=3                   # read sync version
obsidian sync:restore path=note.md version=2                # restore sync version
obsidian sync:deleted                                       # deleted sync files
obsidian sync:open path=note.md                             # open sync history
```

## Plugins

```bash
obsidian plugins                                            # all installed plugins
obsidian plugins filter=community versions                  # community plugins with versions
obsidian plugins:enabled                                    # enabled plugins
obsidian plugins:restrict on                                # enable restricted mode
obsidian plugins:restrict                                   # check restricted mode status
obsidian plugin id=obsidian-map-view                        # plugin info
obsidian plugin:install id=obsidian-git enable              # install + enable
obsidian plugin:enable id=obsidian-git                      # enable plugin
obsidian plugin:disable id=obsidian-git                     # disable plugin
obsidian plugin:reload id=my-plugin                         # reload (dev)
obsidian plugin:uninstall id=obsidian-git                   # uninstall
```

Plugin lifecycle: `list → install → enable → disable → uninstall`. Use `filter=core|community` to narrow lists.

## Themes

```bash
obsidian themes                                             # installed themes
obsidian theme                                              # current theme info
obsidian theme name=Minimal                                 # theme details
obsidian theme:install name=Minimal enable                  # install + activate
obsidian theme:set name=Minimal                             # activate theme
obsidian theme:uninstall name=Minimal                       # remove theme
```

## Bookmarks

```bash
obsidian bookmarks                                          # list bookmarks
obsidian bookmarks verbose format=json                      # with types (JSON)
obsidian bookmark file=note.md                              # bookmark a file
obsidian bookmark folder=Projects                           # bookmark a folder
obsidian bookmark search="meeting"                          # bookmark a search
obsidian bookmark url=https://example.com title="Example"   # bookmark a URL
obsidian bookmark subpath="#section" file=note.md           # bookmark heading
```

## Tabs

```bash
obsidian tabs                                               # list open tabs
obsidian tabs ids                                           # include tab IDs
obsidian tab:open file=note.md                              # open in new tab
obsidian tab:open file=dashboard.md group=<id>              # open in tab group
obsidian tab:open view=graph-view                           # open a view type
```

## Bases

```bash
obsidian bases                                              # list base files
obsidian base:create file=inventory name="Item 1"           # create base item
obsidian base:create file=inventory view="Grid" open newtab # open after create
obsidian base:query file=inventory                          # query base (JSON)
obsidian base:query file=inventory format=csv               # CSV output
obsidian base:views                                         # views in current base
```

`base:query` supports `format=json|csv|tsv|md|paths`.

## Snippets

```bash
obsidian snippets                                           # list CSS snippets
obsidian snippets:enabled                                   # enabled snippets
obsidian snippet:enable name=my-style                       # enable snippet
obsidian snippet:disable name=my-style                      # disable snippet
```

## Dev tools

```bash
obsidian eval code='console.log("hello")'                   # execute JS
obsidian devtools                                           # toggle dev tools
obsidian dev:dom selector=".markdown-source-view"           # query DOM
obsidian dev:dom selector="h1" text                         # text content
obsidian dev:dom selector=".nav-file" all attr=data-path    # attribute values
obsidian dev:css selector=".markdown-preview-view" prop=color  # inspect CSS
obsidian dev:console                                         # console messages
obsidian dev:console level=error limit=10                    # filter + limit
obsidian dev:console clear                                  # clear buffer
obsidian dev:errors                                         # captured errors
obsidian dev:errors clear                                   # clear error buffer
obsidian dev:screenshot path=~/Desktop/shot.png              # screenshot
obsidian dev:debug on                                       # attach CDP debugger
obsidian dev:debug off                                      # detach
obsidian dev:cdp method=Target.getTargets                   # CDP command
obsidian dev:cdp method=Page.navigate params='{"url":"..."}' # CDP with params
obsidian dev:mobile on                                      # mobile emulation
obsidian dev:mobile off                                     # disable
```

## Misc commands

```bash
obsidian vault                                              # vault info
obsidian vault info=files                                   # file count
obsidian vaults                                             # known vaults
obsidian vaults verbose                                     # with paths
obsidian reload                                             # reload vault
obsidian restart                                            # restart app
obsidian version                                            # Obsidian version
obsidian random                                             # open random note
obsidian random folder=Projects                             # from folder
obsidian random:read                                        # read random note
obsidian recents                                            # recently opened
obsidian outline                                            # active file headings
obsidian outline path=note.md format=json                   # headings as JSON
obsidian aliases file=note.md                               # aliases for file
obsidian aliases active                                     # active file aliases
obsidian wordcount path=note.md                             # word + char count
obsidian wordcount path=note.md words                       # words only
obsidian diff path=note.md                                  # local/sync diff
obsidian diff path=note.md from=1 to=3                      # version range
obsidian commands                                           # all commands
obsidian commands filter=editor:                            # filtered
obsidian command id=editor:insert-link                      # execute command
obsidian hotkeys                                            # all hotkeys
obsidian hotkey id=editor:insert-link verbose               # specific hotkey
obsidian workspace                                          # workspace tree
obsidian workspace ids                                      # with item IDs
```

## Tips

- **Quoting**: always quote values with spaces: `name="My Note"`, `query="search phrase"`.
- **Escapes**: `\n` for newline, `\t` for tab in content values.
- **Format flags**: most list commands accept `format=json|tsv|csv`. JSON is best for scripting.
- **Active file default**: most file commands default to the active file when `file`/`path` is omitted.
- **Counts**: pass `total` to get counts instead of detailed output.
- **Vault targeting**: single-vault users can omit `vault=<name>`. Multi-vault users must specify it.
- **Help**: `obsidian help <command>` for syntax on any command.
- **File vs path**: `file=<name>` does wikilink resolution (fuzzy, by note name). `path=<path>` is a literal filesystem path.
