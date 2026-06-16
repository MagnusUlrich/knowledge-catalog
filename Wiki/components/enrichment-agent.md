---
type: reference
title: Knowledge Catalog Enrichment Agent
description: Production-grade metadata enrichment agent using Google ADK, Vertex AI, and Dataplex — generates kcmd mdcode artifacts from BigQuery tables, Google Drive documents, and GitHub repositories.
source_files:
  - agents/enrichment/README.md
updated: 2026-06-16
---

# Knowledge Catalog Enrichment Agent

Source directory: `agents/enrichment/`

A command-line agent that generates **Metadata as Code (mdcode)** for
Google Cloud Knowledge Catalog (Dataplex). It extracts information from source
material and produces the YAML and Markdown artifacts that describe data
assets, ready to be pushed to the catalog with the [`kcmd`](/Wiki/components/kcmd.md) tool.

The agent talks to the catalog **only through `kcmd`** — it never calls the
Dataplex API directly. It runs the read-only `kcmd init` / `kcmd pull` commands
to scaffold and pull existing entries; the user runs `kcmd push` to publish.

## Modes

| Mode | What it does |
|------|-------------|
| **`table`** | Pulls a BigQuery dataset's tables via `kcmd`, routes Google Drive documents to each table by relevance, and writes enriched overviews and `queries` aspect sidecars. Optionally maps columns to Dataplex glossary terms. |
| **`doc`** | Crawls Google Docs (and an optional Drive folder), map-reduce summarizes them, and emits a knowledge-base mdcode snapshot. |
| **`context_overlay`** | Pulls 1P BigQuery table entries via `kcmd reference` (read-only) and creates new context-overlay entries per table in an editable entry group. Avoids touching the live `@bigquery` entries. |

Any mode can optionally ingest:

- **User-feedback proposals** (`--feedback_dir` / `--feedback_files`): treated
  as the highest-priority context source, overriding conflicting information
  from docs, usage signals, or INFORMATION_SCHEMA patterns.
- **GitHub source-code repository** (`--repo`): the agent explores the repo
  agentically via the GitHub MCP server and distills it into code component
  cards. In `doc` mode these surface as distinct knowledge-base entries; in
  `table` / `context_overlay` mode they join the relevance router's candidate
  pool.

After a run, output can be refined with free-text instructions via an
interactive REPL (`--interactive`) or a single re-invocation
(`--refine_instruction`). Refinement reuses already-loaded context and never
re-reads source docs.

## Source layout

```
agents/enrichment/
├── src/
│   ├── agent_runner.py          # CLI entrypoint: flags + dispatch to a mode
│   ├── engine.py                # LLM agents (Vertex Gemini) for all modes
│   ├── common.py                # Shared helpers (run_text, mdcode parsing, trajectory)
│   ├── refine.py                # Multi-turn refinement (REPL + persist/re-invoke)
│   ├── linking.py               # Glossary column-to-term linking (table mode)
│   ├── modes/
│   │   ├── doc_mode.py
│   │   ├── table_mode.py
│   │   └── context_overlay_mode.py
│   └── tools/
│       ├── kcmd_tools.py        # kcmd init/pull/reference + entry reading
│       ├── drive_tools.py       # Google Drive/Docs fetch helpers
│       ├── bq_usage_tools.py    # INFORMATION_SCHEMA query history + queries-aspect sidecar
│       ├── feedback_tools.py    # User-feedback proposal loader + per-table router
│       └── github_tools.py      # GitHub MCP server integration
└── eval/                        # Evaluation CLI
    ├── __main__.py
    ├── dynamic_eval.py          # Golden-free scoring
    ├── golden_eval.py           # Golden-based scoring
    ├── aggregate.py             # Multi-run roll-up
    ├── runner.py                # Case runner
    ├── metrics.py               # Deterministic + LLM-judge metrics
    ├── loaders.py               # catalog/ + trajectory.json readers
    ├── goldens/                 # Golden schema and ready goldens
    └── corpora/                 # Local markdown corpora
```

## Prerequisites

- **Node.js + npm** (any recent LTS) — to build `kcmd`.
- **`gcloud` CLI** — for Application Default Credentials.
- **Python 3.11+**

Setup:

```bash
# Build kcmd
cd agents/mdcode
npm install && npm run build
echo "export PATH=\"$(pwd)/dist:\$PATH\"" >> ~/.bashrc && source ~/.bashrc
cd ../..

# Python deps
python3 -m venv ~/.venv/kc-enrich
source ~/.venv/kc-enrich/bin/activate
pip install -r agents/enrichment/src/requirements.txt

# ADC (includes Drive scope)
gcloud auth application-default login \
  --scopes='openid,https://www.googleapis.com/auth/cloud-platform,https://www.googleapis.com/auth/drive.readonly'
```

## Usage

```bash
export PYTHONPATH=agents/enrichment/src

# Table mode
python3 agents/enrichment/src/agent_runner.py \
  --mode=table \
  --dataset=<project>.<dataset> \
  --folders=<drive_folder_id_or_url> \
  --topic="<instruction>" \
  --project=<gcp_project> \
  --location=<vertex_location> \
  --model=<vertex_model> \
  --output_dir=<local_output_dir>

# Doc mode
python3 agents/enrichment/src/agent_runner.py \
  --mode=doc \
  --docs="https://docs.google.com/document/d/<id>" \
  --entry_group=<project>.<location>.<entryGroupId> \
  --project=<gcp_project> --model=<vertex_model> --output_dir=<out>

# Context-overlay mode
python3 agents/enrichment/src/agent_runner.py \
  --mode=context_overlay \
  --dataset=<project>.<dataset> \
  --entry_group=<project>.<location>.<entryGroupId> \
  --project=<gcp_project> --model=<vertex_model> --output_dir=<out>
```

Required flags in every mode: `--project`, `--model`, `--output_dir`.

## Key flags (selection)

| Flag | Modes | Description |
|------|-------|-------------|
| `--dataset` | table, context_overlay | BigQuery dataset as `project.dataset` |
| `--entry_group` | doc, context_overlay | Dataplex entry group `project.location.groupId` |
| `--topic` | all | Free-text instruction that steers enrichment |
| `--folders` | all | Comma-separated Drive folder IDs/URLs or local `.md` directories |
| `--docs` | doc, context_overlay | Comma-separated Google Doc URLs/IDs or local `.md` files |
| `--include_usage` | table, context_overlay | Fetch BQ INFORMATION_SCHEMA query history (default true) |
| `--glossaries` | table | Map columns to Dataplex glossary terms |
| `--repo` | all | GitHub repo `owner/name` for code context via MCP |
| `--interactive` | all | Start a refinement REPL after the initial run |
| `--refine_instruction` | all | Apply a single refinement turn to an existing output dir |

## Local Markdown inputs

Both `--docs` and `--folders` accept local `.md` files alongside Google
Drive sources. Routing priority:

1. `http://`/`https://` URL — Google Drive
2. Ends in `.md`/`.markdown` — local markdown file
3. Path-shaped string — local directory (read recursively) or file
4. Bare relative name that exists on disk — local
5. Otherwise — Google Drive ID

## Output

The agent writes a `kcmd` mdcode tree into `--output_dir`:

- `catalog.yaml` — manifest written by `kcmd init`
- `catalog/` — per-entry YAML pulled by `kcmd pull` (table mode) or generated
  by the agent (doc mode)
- `<table>.overview.md` sidecars — enriched markdown overviews
- `<table>.queries.md` sidecars — INFORMATION_SCHEMA query patterns (table mode)
- `trajectory.json` — full record of what the agent read and produced

## Evaluation

```bash
cd agents/enrichment
pip install -r eval/requirements.txt
export GOOGLE_CLOUD_PROJECT=<project>
python -m eval --output-dir /tmp/enrich_out
```

Metrics reported (each out of 100):

| Metric | Type | Description |
|--------|------|-------------|
| `structural_validity` | Deterministic | Well-formed mdcode: YAML parses, required fields present, clean Markdown |
| `hallucination_free` | LLM judge | Every factual claim grounded in retrieved source |
| `redundancy_index` | LLM judge | Novel synthesis beyond echoing column names (100 = rich) |
| `disambiguation_efficacy` | LLM judge | Entry clearly distinct from similar entries (100 = distinct) |
| `absence_of_contradictions` | LLM judge | No conflicts within or across entries (100 = none) |

Golden-based eval adds: `concept_recall`, `concept_precision`, `fact_recall`,
`enrichment_diversity`, `entry_grounding`, `persona_alignment`, and more.

## Publishing

The agent only generates mdcode and runs read-only `kcmd` commands.
Publishing to Dataplex is a separate manual step:

```bash
cd /tmp/enrich_out
kcmd push
```

# Citations

[1] [Enrichment agent README](agents/enrichment/README.md)
