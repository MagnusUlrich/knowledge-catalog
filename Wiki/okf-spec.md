---
type: concept
title: Open Knowledge Format (OKF) v0.1
description: Specification summary for OKF — the vendor-neutral markdown-based format for agent-readable knowledge bundles.
source_files:
  - okf/SPEC.md
updated: 2026-06-16
---

# Open Knowledge Format (OKF) v0.1

OKF is an open, human- and agent-friendly format for representing knowledge:
the metadata, context, and curated insight that surrounds data and systems.

The full specification lives at [`okf/SPEC.md`](/okf/SPEC.md).

## Core design

OKF is intentionally minimal: a directory of markdown files with YAML
frontmatter. No schema registry, no central authority, no required tooling.

Design goals:

- **Readable** by humans without tooling.
- **Parseable** by agents without bespoke SDKs.
- **Diffable** in version control.
- **Portable** across tools, organizations, and time.

## Key concepts

| Term | Definition |
|------|------------|
| **Knowledge Bundle** | A self-contained, hierarchical collection of knowledge documents. The unit of distribution. |
| **Concept** | A single unit of knowledge, represented as one markdown file. May describe a data asset, an API, a metric, a playbook, or any abstract idea. |
| **Concept ID** | The file path relative to the bundle root, without the `.md` suffix. Example: `tables/users`. |
| **Frontmatter** | YAML metadata block delimited by `---` at the top of a markdown file. |
| **Body** | Everything in the file after the frontmatter. Standard markdown. |
| **Link** | A standard markdown link from one concept to another, expressing a relationship. |
| **Citation** | A link from a concept to an external source supporting a claim in the body. |

## Bundle structure

```
path/to/bundle/
├── index.md                  # Optional. Directory listing for progressive disclosure.
├── log.md                    # Optional. Chronological history of updates.
├── <concept>.md              # A concept at the bundle root.
└── <subdirectory>/
    ├── index.md
    ├── <concept>.md
    └── <subdirectory>/
        └── ...
```

Bundles may be distributed as a git repository (recommended), a tarball, or a
subdirectory within a larger repository.

### Reserved filenames

| Filename | Purpose |
|----------|---------|
| `index.md` | Directory listing. No frontmatter. |
| `log.md` | Update history. No frontmatter. Entries newest-first, grouped by `YYYY-MM-DD` headings. |

All other `.md` files are concept documents.

## Frontmatter schema

```yaml
---
type: <Type name>                  # REQUIRED
title: <Optional display name>
description: <Optional one-line summary>
resource: <Optional canonical URI for the underlying asset>
tags: [<tag>, <tag>, ...]          # Optional
timestamp: <ISO 8601 datetime>     # Optional last-modified time
# ... other producer-defined key/value pairs
---
```

The only required field is `type`. Producers choose type values freely — they
are not registered centrally. Consumers must tolerate unknown types gracefully.

Recommended fields in priority order: `title`, `description`, `resource`,
`tags`, `timestamp`.

## Conventional body sections

| Heading | Purpose |
|---------|---------|
| `# Schema` | Structured description of columns or fields. |
| `# Examples` | Concrete usage examples, often fenced code blocks. |
| `# Citations` | Numbered external sources backing claims in the body. |

## Cross-linking

Links from one concept to another use standard markdown syntax.

- **Bundle-relative (recommended):** `[customers table](/tables/customers.md)` — starts with `/`, resolved from bundle root.
- **Relative:** `[neighboring concept](./other.md)` — standard relative path.

Consumers build a directed graph from all links. Broken links (target not in
the bundle) are not errors — they represent not-yet-written knowledge.

## Index files

`index.md` files support progressive disclosure. Format:

```markdown
# Section Heading

* [Title 1](relative-url-1) - short description
* [Title 2](relative-url-2) - short description
```

No frontmatter. Descriptions should match the linked concept's `description`
frontmatter field.

## Log files

`log.md` records change history. Format: flat list, newest-first,
grouped under `## YYYY-MM-DD` headings. The leading bold word
(`**Update**`, `**Creation**`, `**Deprecation**`) is a convention, not a
requirement.

## Conformance (v0.1)

A bundle is conformant if:

1. Every non-reserved `.md` file contains a parseable YAML frontmatter block.
2. Every frontmatter block contains a non-empty `type` field.
3. `index.md` and `log.md` follow their defined structures when present.

Consumers must not reject bundles for missing optional fields, unknown `type`
values, unknown extra frontmatter keys, broken cross-links, or missing
`index.md` files.

## Relationship to other formats

OKF is close to:

- LLM "wiki" repositories using markdown + frontmatter as agent-readable knowledge bases.
- Personal knowledge tools like Obsidian and Notion.
- "Metadata as code" approaches storing catalog metadata alongside source code.

OKF differs in being **specified** — it pins down the small set of rules needed
for interoperability without dictating tooling or infrastructure.

## Versioning

Current version: **0.1**. Minor version bumps add backward-compatible
features; major version bumps may make breaking changes. Bundles may declare
their target version with `okf_version: "0.1"` in the root `index.md` frontmatter.

# Citations

[1] [OKF v0.1 Specification](okf/SPEC.md)
