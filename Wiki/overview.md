---
type: reference
title: Repository Overview
description: What the knowledge-catalog fork is, why it was forked, and which parts are relevant.
source_files:
  - README.md
  - okf/SPEC.md
  - okf/README.md
updated: 2026-06-16
---

# Repository Overview

## What it is

`knowledge-catalog` is a Google Cloud repository containing tools, agents, and
samples that demonstrate [Google Cloud Knowledge Catalog](https://cloud.google.com/products/knowledge-catalog)
(formerly Dataplex). It provides:

- A proposed open interchange format for knowledge metadata (OKF)
- A standalone Python enrichment agent that produces OKF bundles from BigQuery
- A production-grade enrichment agent that writes to Dataplex via the `kcmd` CLI
- The `kcmd` TypeScript CLI itself (Metadata as Code tool)
- Sample pipelines and a dataset discovery agent

The upstream repository is
[GoogleCloudPlatform/knowledge-catalog](https://github.com/GoogleCloudPlatform/knowledge-catalog).
This fork was created on 2026-06-16.

## Why it was forked

This fork was retained as a **reference resource**. The repository contains:

1. The OKF specification (`okf/SPEC.md`) — a vendor-neutral, markdown-based
   format for knowledge representation that is relevant to any agent-readable
   catalog or wiki project.
2. Concrete reference implementations showing how enrichment agents produce and
   consume structured knowledge documents.
3. The `kcmd` CLI, which is the official Metadata as Code tool for Dataplex and
   serves as a model for catalog synchronization tooling.

No active development is planned in this fork. It is not a working project —
it is upstream reference material.

## What is relevant

### High relevance

- **`okf/SPEC.md`** — The OKF v0.1 specification. Defines the format that this
  Wiki itself follows (frontmatter schema, bundle structure, index and log
  conventions). Read this first.
- **`okf/README.md`** — Explains the rationale for OKF and documents the
  standalone enrichment agent as a proof-of-concept producer.
- **`agents/mdcode/README.md`** — Full reference for the `kcmd` Metadata as Code
  CLI, including all modes, manifest schema, and MCP server integration.

### Moderate relevance

- **`agents/enrichment/README.md`** — Full enrichment agent using Google ADK +
  Vertex AI. Shows how to wire BigQuery, Google Drive, and Dataplex together for
  production-grade metadata enrichment.
- **`toolbox/`** — TypeScript toolbox libraries for enrichment and mdcode;
  library counterparts to the CLI and agent.

### Lower relevance

- **`samples/`** — Sample pipelines (discovery agent, enrichment samples).
  Useful for concrete examples but not architectural reference.

## Repository layout

```
knowledge-catalog/
├── okf/                     # OKF spec + standalone Python enrichment agent
│   ├── SPEC.md              # The OKF v0.1 specification
│   ├── README.md            # Agent docs and OKF rationale
│   └── pyproject.toml       # Python package
├── agents/
│   ├── enrichment/          # Full enrichment agent (ADK + Vertex AI + Dataplex)
│   └── mdcode/              # kcmd TypeScript CLI
├── toolbox/                 # TypeScript library implementations
├── samples/                 # Sample agents and pipelines
├── README.md                # Repository landing page
└── Wiki/                    # This OKF bundle (fork-local, not upstream)
```

## License

All upstream code is licensed under Apache 2.0. See `LICENSE.md`.
This repository is not an official Google product.
