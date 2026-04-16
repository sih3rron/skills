---
name: miroctl-cli
description: Reference for using miroctl — the Miro CLI tool — to interact with the Miro API from the terminal. Use when the user wants to run miroctl commands, script Miro operations, set up auth/config, or chain API calls via the CLI. Complements the working-with-miro-mcp skill (which covers the MCP/Claude interface); this skill covers direct terminal-based access.
---

# miroctl CLI Reference

## What is miroctl?

`miroctl` is a CLI tool for the Miro API. It gives direct terminal access to boards, items, teams, connectors, and enterprise features — useful for scripting, automation, debugging, and operations that aren't covered by the Miro MCP tools.

Docs: https://vibelab.miro.tools/miroctl/

---

## Global Options

These flags apply to every command:

| Flag | Description |
|------|-------------|
| `--profile <PROFILE>` | Use a named config profile |
| `--token <TOKEN>` | Override stored token inline |
| `--base-url <BASE_URL>` | Override API base URL |
| `--format <FORMAT>` | Output format: `json` (default), `raw`, `jsonl` |
| `--all` | Auto-paginate (fetch all pages) |
| `-v, --verbose` | Enable verbose output |
| `--retry-unsafe` | Retry non-idempotent requests on failure |

---

## Auth

```bash
miroctl auth set-token <TOKEN>      # Store a token for the active profile
miroctl auth login                  # OAuth2 browser flow
miroctl auth login --no-browser     # Print URL instead of opening browser
miroctl auth refresh                # Force-refresh OAuth token
miroctl auth revoke                 # Remove stored token
miroctl auth status                 # Show token status (not the token itself)
```

**OAuth login options:** `--client-id`, `--client-secret`, `--scopes`, `--listen-port` (default: 9898)

---

## Config

```bash
miroctl config init                        # Initialise a new config file
miroctl config list-profiles               # List all profiles
miroctl config use <NAME>                  # Switch active profile
miroctl config get <KEY>                   # Get a config value
miroctl config set <KEY> <VALUE>           # Set a config value
miroctl config show                        # Show config (secrets redacted)
miroctl config show --show-secrets         # Show config including secrets (TTY only)
```

**Key format examples:** `active_profile`, `profiles.default.base_url`

---

## Boards

```bash
miroctl boards list                        # List boards
miroctl boards get                         # Get a specific board
miroctl boards create                      # Create a board
miroctl boards update-board               # Update board
miroctl boards delete                      # Delete board
miroctl boards copy-board                 # Copy a board
```

---

## Items (Generic)

```bash
miroctl items get-items                    # List all items on a board
miroctl items get-items-within-frame       # List items inside a frame
miroctl items get                          # Get a specific item
miroctl items update                       # Update item position or parent
miroctl items delete-item                  # Delete item
```

---

## Item Types

Each item type follows the same CRUD pattern: `create`, `get`, `update`, `delete`.

| Command Group | Item Type |
|---------------|-----------|
| `miroctl sticky-notes` | Sticky notes |
| `miroctl cards` | Cards |
| `miroctl shapes` | Shapes |
| `miroctl flowchart-shapes` | Flowchart shapes (experimental) |
| `miroctl texts` | Text items |
| `miroctl frames` | Frames |
| `miroctl images` | Images (local file or URL) |
| `miroctl documents` | Documents (local file or URL) |
| `miroctl embeds` | Embeds |
| `miroctl app-cards` | App cards |
| `miroctl mind-map-nodes` | Mind map nodes |

**Image/document creation variants:**
```bash
miroctl images create-image-item-using-url
miroctl images create-image-item-using-local-file
miroctl documents create-document-item-using-url
miroctl documents create-document-item-using-file-from-device
```

---

## Connectors

```bash
miroctl connectors list
miroctl connectors get
miroctl connectors create
miroctl connectors update
miroctl connectors delete
```

---

## Tags

```bash
miroctl tags create
miroctl tags get
miroctl tags update
miroctl tags delete
miroctl tags attach                        # Attach tag to item
miroctl tags remove                        # Remove tag from item
miroctl tags get-tags-from-board
miroctl tags get-tags-from-item
miroctl tags get-items-by-tag
```

---

## Groups

```bash
miroctl groups create
miroctl groups get
miroctl groups get-all-groups
miroctl groups get-items-by-group-id
miroctl groups update                      # Update group with new items
miroctl groups delete-group
miroctl groups un-group                    # Ungroup items
```

---

## Bulk Operations

```bash
miroctl bulk-operations create-items                              # Bulk create items (JSON)
miroctl bulk-operations create-items-in-bulk-using-file-from-device   # Bulk create from file
```

---

## Board Members

```bash
miroctl board-members list
miroctl board-members get
miroctl board-members share
miroctl board-members update
miroctl board-members remove
```

---

## Projects & Members

```bash
miroctl projects list / get / create / update / delete
miroctl project-members list / get / create / update / delete
miroctl project-settings list / update
```

---

## Teams & Members

```bash
miroctl teams list / get / create / update / delete
miroctl team-members list / get / invite / update / delete
miroctl team-settings enterprise-get-team-settings
miroctl team-settings enterprise-get-default-team-settings
miroctl team-settings update
```

---

## Board Export

```bash
miroctl board-export create             # Start an export job
miroctl board-export get                # Get export job status
miroctl board-export list               # Get export results
```

---

## Board Content Logs

```bash
miroctl board-content-logs list         # Retrieve content change logs
```

---

## Tokens & OAuth

```bash
miroctl tokens list                     # Get access token info
miroctl tokens create                   # Revoke token (v1)
miroctl o-auth create                   # Revoke token (v2)
```

---

## Webhooks

```bash
miroctl webhooks list / get / create / update / delete
```

---

## Enterprise

```bash
miroctl organizations get
miroctl organization-members list / get
miroctl audit-logs list
miroctl board-classification enterprise-dataclassification-board-get
miroctl board-classification create                               # Update board classification
miroctl board-classification enterprise-dataclassification-team-boards-bulk
miroctl legal-holds get-all-cases / get-case / get-all-legal-holds / get-legal-hold / get-legal-hold-content-items
miroctl reset-all-sessions-of-a-user reset
```

---

## App Metrics

```bash
miroctl app-metrics get-metrics
miroctl app-metrics get-metrics-total
```

---

## Escape Hatch: Raw API Calls

For any operation not exposed as a named subcommand:

```bash
miroctl api call <OPERATION_ID> \
  --path board_id=<BOARD_ID> \
  --query limit=10 \
  --data '{"key": "value"}' \
  --header "X-Custom:value"
```

Use `--data @file.json` to pass a JSON file as the request body.

---

## Common Patterns

### Paginate through all results
```bash
miroctl boards list --all
miroctl items get-items --all --query board_id=<ID>
```

### Switch between environments
```bash
miroctl config use staging
miroctl boards list
```

### Output as JSON Lines (for piping)
```bash
miroctl boards list --format jsonl | jq '.name'
```

### Pass a token inline (no stored config needed)
```bash
miroctl boards list --token <MY_TOKEN>
```

---

## Relationship to Miro MCP

| Use Case | Prefer |
|----------|--------|
| Creating content on boards during a Claude session | Miro MCP (`working-with-miro-mcp` skill) |
| Scripting, automation, CI/CD | `miroctl` CLI |
| Auth setup and profile management | `miroctl auth` / `miroctl config` |
| Operations not in MCP (bulk create, export, audit logs, legal holds) | `miroctl` CLI |
| Debugging raw API responses | `miroctl api call` |