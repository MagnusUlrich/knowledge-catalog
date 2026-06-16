---
type: reference
title: OKF Standalone Enrichment Agent
description: A standalone Python agent that ingests BigQuery metadata and emits OKF bundles, using Google ADK and Gemini as a proof of concept for OKF production.
source_files:
  - okf/README.md
  - okf/SPEC.md
  - okf/pyproject.toml
updated: 2026-06-16
---

# OKF Standalone Enrichment Agent

Source directory: `okf/`

This agent is a **proof of concept** for producing [OKF](/Wiki/okf-spec.md)
bundles automatically. It is the primary demonstration of the OKF format in
practice — the format is the contribution; this agent makes the format tangible
at both the production end (enrichment) and the consumption end (visualization).

The agent is built on the Google [Agent Development Kit](https://adk.dev/)
with Gemini as the model backend. BigQuery is the first source implementation;
the `Source` interface is designed to be extended to other sources.

## What it does

The agent ingests metadata from a pluggable source and emits an OKF bundle: a
directory of markdown documents with YAML frontmatter that catalog tools,
downstream agents, and humans can all read.

Enrichment runs in **two passes**:

1. **BQ pass** — writes one OKF concept document per concept advertised by the
   source, using BigQuery metadata alone.
2. **Web pass** — the LLM acts as its own crawler. It receives seed URLs,
   fetches them via the `fetch_url` tool, and decides which outbound links to
   follow based on relevance to the existing concepts. For each fetched page the
   agent either (a) enriches an existing concept doc, (b) mints a standalone
   `references/<slug>` doc, or (c) skips. A hard `--web-max-pages` cap and
   same-domain allowed-hosts filter prevent runaway crawling. Use `--no-web` to
   skip the web pass.

## Key features

- **Progressive disclosure** — auto-generated `index.md` files let agents and
  humans navigate the bundle one level at a time.
- **Graph-shaped output** — concepts link to each other via standard markdown
  links, expressing richer relationships than the directory hierarchy alone.
- **Bundle visualizer** — the `visualize` subcommand renders any OKF bundle as
  a self-contained interactive HTML file using Cytoscape.js for graph layout and
  marked for in-browser markdown rendering. No backend required.
- **Version-controllable** — bundles live in git; enrichment runs produce
  diffable artifacts.

## Install

```bash
python3.13 -m venv .venv
.venv/bin/pip install --index-url https://pypi.org/simple/ -e .[dev]
```

Run from the `okf/` directory.

## Credentials

- **BigQuery:** `gcloud auth application-default login` plus a billing project
  (`gcloud config set project <id>`). Public datasets are readable; the caller's
  project is billed for query bytes.
- **Gemini:** Set `GEMINI_API_KEY` (AI Studio) **or** use Vertex AI by setting
  `GOOGLE_GENAI_USE_VERTEXAI=true`, `GOOGLE_CLOUD_PROJECT=<id>`, and
  `GOOGLE_CLOUD_LOCATION=<region>`.

## Run

Minimum invocation — point at a BigQuery dataset and an output directory:

```bash
.venv/bin/python -m enrichment_agent enrich \
    --source bq \
    --dataset <project>.<dataset> \
    --web-seed-file <path/to/seeds.txt> \
    --out ./bundles/<name>
```

Iterate on a single concept with `--concept <type>/<name>` (repeatable).
Omit `--web-seed-file` or pass `--no-web` for a BQ-only run.

## Visualize

```bash
.venv/bin/python -m enrichment_agent visualize --bundle ./bundles/<name>
```

Writes `bundles/<name>/viz.html`. The viewer shows:

- A force-directed graph with colored nodes by concept type and directed edges
  from cross-links in the markdown bodies.
- A detail panel with frontmatter (description, resource, tags) and rendered
  markdown body, with internal links rewired to navigate within the viewer.
- Backlinks list (computed from the reverse link graph).
- Search box (matches title, concept ID, and tags), type filter, and switchable
  graph layouts.

## Sample bundles

Three ready-to-browse bundles are checked into `okf/bundles/`:

| Bundle | Dataset | Notes |
|--------|---------|-------|
| `bundles/ga4/` | GA4 e-commerce | Seeded with canonical GA4 BigQuery Export docs |
| `bundles/stackoverflow/` | Stack Overflow public dataset | Multi-concept enrichment from cross-cutting docs |
| `bundles/crypto_bitcoin/` | Bitcoin blocks/transactions | Cross-table foreign-key relationships in prose |

Each bundle ships a `viz.html` file for immediate browser viewing.

## Tests

```bash
.venv/bin/pytest
```

Test files are under `okf/tests/` and cover the BigQuery source, bundle tools,
document handling, index generation, viewer, and web fetching.

# Citations

[1] [OKF standalone agent README](okf/README.md)
[2] [OKF v0.1 Specification](okf/SPEC.md)
