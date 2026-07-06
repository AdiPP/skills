# Obsidian CLI — Full Command Reference

Reference for every `obsidian` CLI command. Commands grouped by category, same order as `SKILL.md`.

## File operations

### `obsidian create`

Create a new file.

```
obsidian create name=<name> [content=<text>] [template=<name>] [overwrite] [open] [newtab]
obsidian create path=<path> [content=<text>] [template=<name>] [overwrite] [open] [newtab]
```

| Param | Required | Description |
|---|---|---|
| `name=<name>` | one of name/path | File name (wikilink resolution) |
| `path=<path>` | one of name/path | Exact file path |
| `content=<text>` | no | Initial content |
| `template=<name>` | no | Template to apply |
| `overwrite` | no | Overwrite if file exists |
| `open` | no | Open file after creating |
| `newtab` | no | Open in new tab |

### `obsidian read`

Read file contents.

```
obsidian read [file=<name> | path=<path>]
```

Defaults to active file when omitted.

### `obsidian append`

Append content to a file.

```
obsidian append <content=<text>> [file=<name> | path=<path>] [inline]
```

| Param | Required | Description |
|---|---|---|
| `content=<text>` | yes | Content to append. `\n` for newline, `\t` for tab |
| `file=<name>` | no | File name |
| `path=<path>` | no | File path |
| `inline` | no | Append without trailing newline |

### `obsidian prepend`

Prepend content to a file. Same params as `append`.

```
obsidian prepend <content=<text>> [file=<name> | path=<path>] [inline]
```

### `obsidian delete`

Move file to trash or delete permanently.

```
obsidian delete [file=<name> | path=<path>] [permanent]
```

`permanent` skips the trash.

### `obsidian move`

Move or rename a file.

```
obsidian move <to=<path>> [file=<name> | path=<path>]
```

`to=<path>` is required — destination folder or full path.

### `obsidian rename`

Rename a file.

```
obsidian rename <name=<name>> [file=<name> | path=<path>]
```

`name=<name>` is the new file name (required).

### `obsidian open`

Open a file in Obsidian.

```
obsidian open [file=<name> | path=<path>] [newtab]
```

### `obsidian file`

Show file info (metadata, frontmatter, character count, etc.).

```
obsidian file [file=<name> | path=<path>]
```

### `obsidian files`

List files in the vault.

```
obsidian files [folder=<path>] [ext=<extension>] [total]
```

| Param | Description |
|---|---|
| `folder=<path>` | Filter by folder |
| `ext=<extension>` | Filter by extension (e.g. `md`, `png`) |
| `total` | Return file count only |

### `obsidian folders`

List folders in the vault.

```
obsidian folders [folder=<path>] [total]
```

| Param | Description |
|---|---|
| `folder=<path>` | Filter by parent folder |
| `total` | Return folder count only |

### `obsidian folder`

Show folder info.

```
obsidian folder <path=<path>> [info=files|folders|size]
```

`path=<path>` is required. `info` returns only that metric.

## Daily notes

### `obsidian daily`

Open today's daily note.

```
obsidian daily [paneType=tab|split|window]
```

### `obsidian daily:read`

Read today's daily note content.

```
obsidian daily:read
```

### `obsidian daily:append`

Append content to daily note.

```
obsidian daily:append <content=<text>> [inline] [open] [paneType=tab|split|window]
```

### `obsidian daily:prepend`

Prepend content to daily note. Same params as `daily:append`.

### `obsidian daily:path`

Get the filesystem path of today's daily note.

```
obsidian daily:path
```

## Search

### `obsidian search`

Search vault contents.

```
obsidian search <query=<text>> [path=<folder>] [limit=<n>] [case] [total] [format=text|json]
```

| Param | Required | Description |
|---|---|---|
| `query=<text>` | yes | Search query |
| `path=<folder>` | no | Limit search to folder |
| `limit=<n>` | no | Max files to return |
| `case` | no | Case-sensitive search |
| `total` | no | Match count only |
| `format=text|json` | no | Output format (default: text) |

### `obsidian search:context`

Search with matching line context. Same flags as `search` (minus `format`).

```
obsidian search:context <query=<text>> [path=<folder>] [limit=<n>] [case] [format=text|json]
```

### `obsidian search:open`

Open Obsidian search view with initial query.

```
obsidian search:open [query=<text>]
```

## Tags

### `obsidian tags`

List tags in the vault.

```
obsidian tags [file=<name> | path=<path>] [total] [counts] [sort=count] [format=json|tsv|csv] [active]
```

| Param | Description |
|---|---|
| `file=<name>` | Tags for a specific file |
| `path=<path>` | Tags for a specific path |
| `total` | Tag count only |
| `counts` | Include occurrence counts |
| `sort=count` | Sort by count (default: name) |
| `format` | Output format (default: tsv) |
| `active` | Tags for the active file |

### `obsidian tag`

Get info for a specific tag.

```
obsidian tag <name=<tag>> [total] [verbose]
```

| Param | Description |
|---|---|
| `name=<tag>` | Tag name (required) |
| `total` | Occurrence count only |
| `verbose` | Include file list and count |

## Properties

### `obsidian properties`

List properties in the vault.

```
obsidian properties [file=<name> | path=<path>] [name=<name>] [total] [sort=count] [counts] [format=yaml|json|tsv] [active]
```

| Param | Description |
|---|---|
| `file=<name>` | Properties for a file |
| `path=<path>` | Properties for a path |
| `name=<name>` | Specific property count |
| `total` | Property count only |
| `sort=count` | Sort by count (default: name) |
| `counts` | Include occurrence counts |
| `format` | Output format (default: yaml) |
| `active` | Properties for active file |

### `obsidian property:read`

Read a property value from a file.

```
obsidian property:read <name=<name>> [file=<name> | path=<path>]
```

`name=<name>` is required.

### `obsidian property:set`

Set a property on a file.

```
obsidian property:set <name=<name>> <value=<value>> [type=text|list|number|checkbox|date|datetime] [file=<name> | path=<path>]
```

| Param | Required | Description |
|---|---|---|
| `name=<name>` | yes | Property name |
| `value=<value>` | yes | Property value |
| `type=<type>` | no | Property type (default: text) |
| `file`/`path` | no | Target file |

### `obsidian property:remove`

Remove a property from a file.

```
obsidian property:remove <name=<name>> [file=<name> | path=<path>]
```

## Tasks

### `obsidian tasks`

List tasks in the vault.

```
obsidian tasks [file=<name> | path=<path>] [total] [done] [todo] [status="<char>"] [verbose] [format=json|tsv|csv] [active] [daily]
```

| Param | Description |
|---|---|
| `file=<name>` | Filter by file |
| `path=<path>` | Filter by path |
| `total` | Task count only |
| `done` | Show completed tasks |
| `todo` | Show incomplete tasks |
| `status="<char>"` | Filter by status character |
| `verbose` | Group by file with line numbers |
| `format` | Output format (default: text) |
| `active` | Active file tasks |
| `daily` | Daily note tasks |

### `obsidian task`

Show or update a specific task.

```
obsidian task [ref=<path:line>] [file=<name> | path=<path>] [line=<n>] [toggle|done|todo|status="<char>"] [daily]
```

| Param | Description |
|---|---|
| `ref=<path:line>` | Task reference (e.g. `note.md:42`) |
| `file=<name>` | File name |
| `path=<path>` | File path |
| `line=<n>` | Line number |
| `toggle` | Toggle task status |
| `done` | Mark as done |
| `todo` | Mark as todo |
| `status="<char>"` | Set custom status character |
| `daily` | Use daily note |

## Templates

### `obsidian templates`

List available templates.

```
obsidian templates [total]
```

### `obsidian template:read`

Read template content.

```
obsidian template:read <name=<template>> [resolve] [title=<title>]
```

| Param | Description |
|---|---|
| `name=<template>` | Template name (required) |
| `resolve` | Resolve template variables (e.g. `{{title}}`) |
| `title=<title>` | Title for variable resolution |

### `obsidian template:insert`

Insert template into the active file.

```
obsidian template:insert <name=<template>>
```

## Links and backlinks

### `obsidian backlinks`

List backlinks (incoming links) to a file.

```
obsidian backlinks [file=<name> | path=<path>] [counts] [total] [format=json|tsv|csv]
```

| Param | Description |
|---|---|
| `file`/`path` | Target file (default: active file) |
| `counts` | Include link counts |
| `total` | Backlink count only |
| `format` | Output format (default: tsv) |

### `obsidian links`

List outgoing links from a file.

```
obsidian links [file=<name> | path=<path>] [total]
```

### `obsidian orphans`

List files with no incoming links.

```
obsidian orphans [total] [all]
```

`all` includes non-markdown files.

### `obsidian deadends`

List files with no outgoing links.

```
obsidian deadends [total] [all]
```

### `obsidian unresolved`

List broken wikilinks (links to files that don't exist).

```
obsidian unresolved [total] [counts] [verbose] [format=json|tsv|csv]
```

| Param | Description |
|---|---|
| `total` | Unresolved link count |
| `counts` | Include link counts |
| `verbose` | Include source files |
| `format` | Output format (default: tsv) |

## History

### `obsidian history`

List version history for a file.

```
obsidian history [file=<name> | path=<path>]
```

### `obsidian history:list`

List files that have version history.

```
obsidian history:list
```

### `obsidian history:read`

Read a specific version of a file.

```
obsidian history:read [file=<name> | path=<path>] [version=<n>]
```

`version` defaults to 1 (oldest).

### `obsidian history:restore`

Restore a file to a previous version.

```
obsidian history:restore [file=<name> | path=<path>] <version=<n>>
```

`version=<n>` is required.

### `obsidian history:open`

Open the file recovery view for a file.

```
obsidian history:open [file=<name> | path=<path>]
```

## Sync

### `obsidian sync`

Pause or resume sync.

```
obsidian sync on|off
```

### `obsidian sync:status`

Show sync status.

```
obsidian sync:status
```

### `obsidian sync:history`

List sync version history for a file.

```
obsidian sync:history [file=<name> | path=<path>] [total]
```

### `obsidian sync:read`

Read a specific sync version.

```
obsidian sync:read [file=<name> | path=<path>] <version=<n>>
```

`version=<n>` is required.

### `obsidian sync:restore`

Restore a sync version.

```
obsidian sync:restore [file=<name> | path=<path>] <version=<n>>
```

### `obsidian sync:deleted`

List deleted files in sync.

```
obsidian sync:deleted [total]
```

### `obsidian sync:open`

Open sync history for a file.

```
obsidian sync:open [file=<name> | path=<path>]
```

## Plugins

### `obsidian plugins`

List installed plugins.

```
obsidian plugins [filter=core|community] [versions] [format=json|tsv|csv]
```

### `obsidian plugins:enabled`

List enabled plugins. Same flags as `plugins`.

```
obsidian plugins:enabled [filter=core|community] [versions] [format=json|tsv|csv]
```

### `obsidian plugins:restrict`

Toggle or check restricted mode.

```
obsidian plugins:restrict [on|off]
```

With no `on`/`off`, reports current restricted mode status.

### `obsidian plugin`

Get plugin info.

```
obsidian plugin <id=<plugin-id>>
```

### `obsidian plugin:install`

Install a community plugin.

```
obsidian plugin:install <id=<id>> [enable]
```

`enable` activates the plugin after install.

### `obsidian plugin:enable`

Enable a plugin.

```
obsidian plugin:enable <id=<id>> [filter=core|community]
```

### `obsidian plugin:disable`

Disable a plugin.

```
obsidian plugin:disable <id=<id>> [filter=core|community]
```

### `obsidian plugin:reload`

Reload a plugin (for development).

```
obsidian plugin:reload <id=<id>>
```

### `obsidian plugin:uninstall`

Uninstall a community plugin.

```
obsidian plugin:uninstall <id=<id>>
```

## Themes

### `obsidian themes`

List installed themes.

```
obsidian themes [versions]
```

### `obsidian theme`

Show active theme or get info on a specific theme.

```
obsidian theme [name=<name>]
```

### `obsidian theme:install`

Install a community theme.

```
obsidian theme:install <name=<name>> [enable]
```

`enable` activates the theme after install.

### `obsidian theme:set`

Set the active theme.

```
obsidian theme:set <name=<name>>
```

Pass `name=` (empty) to revert to default theme.

### `obsidian theme:uninstall`

Uninstall a theme.

```
obsidian theme:uninstall <name=<name>>
```

## Bookmarks

### `obsidian bookmarks`

List bookmarks.

```
obsidian bookmarks [total] [verbose] [format=json|tsv|csv]
```

| Param | Description |
|---|---|
| `total` | Bookmark count only |
| `verbose` | Include bookmark types (file, folder, search, url) |
| `format` | Output format (default: tsv) |

### `obsidian bookmark`

Add a bookmark.

```
obsidian bookmark [file=<path>] [subpath=<subpath>] [folder=<path>] [search=<query>] [url=<url>] [title=<title>]
```

| Param | Description |
|---|---|
| `file=<path>` | File path to bookmark |
| `subpath=<subpath>` | Heading or block within file |
| `folder=<path>` | Folder to bookmark |
| `search=<query>` | Search query to bookmark |
| `url=<url>` | URL to bookmark |
| `title=<title>` | Bookmark title |

At least one of `file`, `folder`, `search`, or `url` is required.

## Tabs

### `obsidian tabs`

List open tabs.

```
obsidian tabs [ids]
```

`ids` includes tab group IDs.

### `obsidian tab:open`

Open a new tab.

```
obsidian tab:open [file=<path>] [group=<id>] [view=<type>]
```

| Param | Description |
|---|---|
| `file=<path>` | File to open |
| `group=<id>` | Tab group ID |
| `view=<type>` | View type to open (e.g. `graph-view`) |

## Bases

Obsidian Bases (formerly Database Folders) — structured data with views.

### `obsidian bases`

List all base files in vault.

```
obsidian bases
```

### `obsidian base:create`

Create a new item in a base.

```
obsidian base:create [file=<name> | path=<path>] [view=<name>] [name=<name>] [content=<text>] [open] [newtab]
```

| Param | Description |
|---|---|
| `file=<name>` | Base file name |
| `path=<path>` | Base file path |
| `view=<name>` | View name |
| `name=<name>` | New item name |
| `content=<text>` | Initial content |
| `open` | Open file after creating |
| `newtab` | Open in new tab |

### `obsidian base:query`

Query a base and return results.

```
obsidian base:query [file=<name> | path=<path>] [view=<name>] [format=json|csv|tsv|md|paths]
```

`format` defaults to `json`.

### `obsidian base:views`

List views in the current base file.

```
obsidian base:views
```

## Snippets

### `obsidian snippets`

List installed CSS snippets.

```
obsidian snippets
```

### `obsidian snippets:enabled`

List enabled CSS snippets.

```
obsidian snippets:enabled
```

### `obsidian snippet:enable`

Enable a CSS snippet.

```
obsidian snippet:enable <name=<name>>
```

### `obsidian snippet:disable`

Disable a CSS snippet.

```
obsidian snippet:disable <name=<name>>
```

## Dev tools

### `obsidian eval`

Execute JavaScript in Obsidian's context.

```
obsidian eval <code=<javascript>>
```

`code` is required. Returns the result of the expression.

### `obsidian devtools`

Toggle Electron developer tools.

```
obsidian devtools
```

### `obsidian dev:dom`

Query DOM elements.

```
obsidian dev:dom <selector=<css>> [total] [text] [inner] [all] [attr=<name>] [css=<prop>]
```

| Param | Description |
|---|---|
| `selector=<css>` | CSS selector (required) |
| `total` | Element count only |
| `text` | Return text content |
| `inner` | Return innerHTML instead of outerHTML |
| `all` | Return all matches (default: first only) |
| `attr=<name>` | Get attribute value |
| `css=<prop>` | Get CSS property value |

### `obsidian dev:css`

Inspect CSS with source locations.

```
obsidian dev:css <selector=<css>> [prop=<name>]
```

### `obsidian dev:console`

Show captured console messages.

```
obsidian dev:console [clear] [limit=<n>] [level=log|warn|error|info|debug]
```

`clear` empties the buffer. `limit` defaults to 50.

### `obsidian dev:errors`

Show captured errors.

```
obsidian dev:errors [clear]
```

### `obsidian dev:screenshot`

Take a screenshot of the Obsidian window.

```
obsidian dev:screenshot [path=<filename>]
```

### `obsidian dev:debug`

Attach or detach Chrome DevTools Protocol debugger.

```
obsidian dev:debug on|off
```

### `obsidian dev:cdp`

Run a Chrome DevTools Protocol command.

```
obsidian dev:cdp <method=<CDP.method>> [params=<json>]
```

`method` is required. `params` is a JSON object of CDP method parameters.

### `obsidian dev:mobile`

Toggle mobile emulation.

```
obsidian dev:mobile on|off
```

## Misc commands

### `obsidian vault`

Show vault info.

```
obsidian vault [info=name|path|files|folders|size]
```

### `obsidian vaults`

List known vaults.

```
obsidian vaults [total] [verbose]
```

`verbose` includes vault paths.

### `obsidian reload`

Reload the current vault.

```
obsidian reload
```

### `obsidian restart`

Restart the Obsidian app.

```
obsidian restart
```

### `obsidian version`

Show Obsidian version.

```
obsidian version
```

### `obsidian random`

Open a random note.

```
obsidian random [folder=<path>] [newtab]
```

### `obsidian random:read`

Read a random note.

```
obsidian random:read [folder=<path>]
```

### `obsidian recents`

List recently opened files.

```
obsidian recents [total]
```

### `obsidian outline`

Show headings for a file.

```
obsidian outline [file=<name> | path=<path>] [format=tree|md|json] [total]
```

Defaults to active file. `format` defaults to `tree`.

### `obsidian aliases`

List aliases in the vault.

```
obsidian aliases [file=<name> | path=<path>] [total] [verbose] [active]
```

| Param | Description |
|---|---|
| `file`/`path` | Aliases for a specific file |
| `total` | Alias count only |
| `verbose` | Include file paths |
| `active` | Aliases for active file |

### `obsidian wordcount`

Count words and characters.

```
obsidian wordcount [file=<name> | path=<path>] [words] [characters]
```

Default shows both. `words` or `characters` returns only that metric.

### `obsidian commands`

List available commands.

```
obsidian commands [filter=<prefix>]
```

`filter` narrows by command ID prefix (e.g. `editor:`).

### `obsidian command`

Execute an Obsidian command.

```
obsidian command <id=<command-id>>
```

`id` is the internal command ID (e.g. `editor:insert-link`).

### `obsidian hotkeys`

List hotkeys.

```
obsidian hotkeys [total] [verbose] [format=json|tsv|csv] [all]
```

`all` includes commands without hotkeys.

### `obsidian hotkey`

Get hotkey for a command.

```
obsidian hotkey <id=<command-id>> [verbose]
```

`verbose` shows whether the hotkey is custom or default.

### `obsidian workspace`

Show workspace tree (panes, tabs, groups).

```
obsidian workspace [ids]
```

`ids` includes workspace item IDs.

### `obsidian diff`

List or diff local/sync versions of a file.

```
obsidian diff [file=<name> | path=<path>] [from=<n>] [to=<n>] [filter=local|sync]
```

| Param | Description |
|---|---|
| `file`/`path` | Target file |
| `from=<n>` | Version to diff from |
| `to=<n>` | Version to diff to |
| `filter=local|sync` | Filter by version source |

### `obsidian help`

Show help for any command.

```
obsidian help [<command>]
```
