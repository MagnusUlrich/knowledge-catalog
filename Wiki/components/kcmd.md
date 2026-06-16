---
type: reference
title: kcmd — Metadata as Code CLI
description: TypeScript CLI for bidirectional sync between local YAML/Markdown metadata files and Google Dataplex Knowledge Catalog, with a built-in MCP server for agent integration.
source_files:
  - agents/mdcode/README.md
updated: 2026-06-16
---

# kcmd — Metadata as Code CLI

Source directory: `agents/mdcode/`

`kcmd` (Metadata as Code) is a TypeScript CLI that provides a
developer-friendly workflow for managing Google Cloud Knowledge Catalog
(Dataplex) metadata. Data stewards, data producers, and AI agents author and
enrich metadata as local YAML and Markdown files — just like source code — and
use `kcmd` to sync those files with the Dataplex catalog service.

## Key features

### Core metadata management
- Manage catalog entries as local YAML or Markdown files organized in a
  hierarchy that mirrors cloud resources.
- Bidirectional sync: `kcmd pull` downloads, `kcmd push` uploads.

### Grounding and safe publishing
- `kcmd reference` pulls read-only system metadata (schemas, etc.) into
  `.ref.yaml` files — an authoritative baseline for enrichment that is never
  pushed back.
- `kcmd push` auto-creates missing Entry Groups and Entries from the local
  filesystem during a push.

### Content flexibility
- **YAML Layout** for data assets (BigQuery tables, Entry Groups, BigLake).
- **Markdown Layout** (`--kb` mode) for human-authored Knowledge Bases.
- Markdown sidecars (`<entry>.overview.md`) detach long-form content from YAML
  for cleaner diffs.

### MCP server
Built-in Model Context Protocol server. Exposes `list-entries`, `lookup-entry`,
and `modify-entry` tools, allowing AI agents to interact with the catalog
directly.

### EntryLinks and glossary
- EntryLinks (e.g., `definition` links from BQ columns to glossary terms,
  `schema-join` between tables) are first-class artifacts in `pull` and `push`.
- Column-level links (those carrying a `Schema.<field>` source path) are inlined
  under `aspects.schema.fields[].links` in the table YAML.
- Glossary scope: manage Business Glossaries (terms, categories) using the same
  `pull`/`push` workflow as other source types.

## Initialization modes

`kcmd init` selects a source type (mode) for the workspace. Exactly one primary
source type is required per workspace.

| Mode | Flag | ID format | Target |
|------|------|-----------|--------|
| BigQuery | `--bigquery-dataset` | `project.dataset` | Tables, views, schemas (YAML layout) |
| Knowledge Base | `--kb` | `project.location.entryGroupId` | Human-authored wiki/docs (Markdown layout) |
| Entry Group | `--entry-group` | `project.location.entryGroupId` | Custom catalog entries (YAML layout) |
| BigLake | `--biglake-namespace` | `project.catalog.namespace` | Iceberg/BigLake tables (YAML layout) |
| Glossary | `--glossary` | `project.location.glossaryId` | Business Glossary terms and categories (YAML layout) |

```bash
# BigQuery mode
kcmd init --bigquery-dataset my-project.my-dataset

# Knowledge Base (wiki) mode
kcmd init --kb my-project.us-central1.my-knowledge-base

# Glossary mode — by ID, by display name, or location mode (all glossaries)
kcmd init --glossary my-project.us-central1.my-glossary-id
kcmd init --glossary my-project.us-central1          # all glossaries in location
```

## Sync commands

```bash
# Pull editable metadata into catalog/
kcmd pull

# Pull read-only reference metadata into *.ref.yaml files
kcmd reference

# Push local edits to the Dataplex catalog
kcmd push

# Validate without pushing
kcmd push --validate-only

# Overwrite service metadata, ignoring conflicts
kcmd push --force
```

`.ref.yaml` files are strictly read-only — they are skipped by `kcmd push`.

## MCP server integration

```json
{
  "mcpServers": {
    "kcmd": {
      "command": "npx",
      "args": ["-y", "kcmd", "mcp", "--path", "/absolute/path/to/workspace"]
    }
  }
}
```

Provides tools: `list-entries`, `lookup-entry`, `modify-entry`.

## Catalog manifest (`catalog.yaml`)

The manifest defines synchronization scope, managed types, and optional
reference layers.

```yaml
# Primary resource
scope: bigquery-mode-id.my-project.my-dataset

# What to manage locally
snapshot:
  entries:
    - dataplex-types.global.bigquery-table
  aspects:
    - dataplex-types.global.schema
    - dataplex-types.global.overview
  entryLinks:
    - definition
    - synonym

# What to push back to the service
publishing:
  aspects:
    - dataplex-types.global.overview
  entryLinks:
    - definition

# Read-only reference layer (never pushed)
reference:
  scope: bq-dataset.my-project.my-dataset
  snapshot:
    entries:
      - dataplex-types.global.bigquery-table
    aspects:
      - dataplex-types.global.schema
```

## Directory layout

### YAML layout (data assets)

```
/
├── catalog.yaml
└── catalog/
    └── bigquery/
        └── my-project/
            ├── my-dataset.yaml
            └── my-dataset/
                ├── table1.yaml          # Editable entry
                ├── table1.ref.yaml      # Read-only reference layer
                └── table2.overview.md   # Markdown sidecar
```

### Markdown layout (Knowledge Base)

```
/
├── catalog.yaml
└── catalog/
    └── my-namespace/
        └── my-project/
            └── my-location/
                ├── page1.md
                └── page2.md
```

## Glossary behavior

`kcmd push` **does not create** Glossary, GlossaryCategory, or GlossaryTerm
resources — these are control-plane resources that must be created by humans
via the Dataplex console or `gcloud dataplex glossaries create`. After creation,
`kcmd push` can update descriptions and labels on existing resources, and it
does manage EntryLinks that reference glossary terms.

## Build

```bash
cd agents/mdcode
npm install
npm run build    # output: agents/mdcode/dist/kcmd
npm run test
```

After building, add `agents/mdcode/dist/` to `PATH` to use `kcmd` directly.
The [`enrichment-agent`](enrichment-agent.md) also auto-discovers the binary at
`agents/mdcode/dist/kcmd` (override with `$KCMD_BIN`).

## Authentication

```bash
gcloud auth application-default login
```

Both the CLI and MCP server use `gcloud` for authentication tokens.

# Citations

[1] [kcmd README](agents/mdcode/README.md)
