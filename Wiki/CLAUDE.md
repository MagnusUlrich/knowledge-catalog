# CLAUDE.md — knowledge-catalog Wiki

## What this wiki is

This Wiki is an OKF knowledge bundle documenting the
`knowledge-catalog` repository — a fork of
[GoogleCloudPlatform/knowledge-catalog](https://github.com/GoogleCloudPlatform/knowledge-catalog)
forked on 2026-06-16.

The repository is **reference material**, not an actively developed project.
All source code here is upstream Google Cloud code. The wiki exists to make
that upstream code navigable and useful as a knowledge reference.

---

## Ingest guidance

When reading or reasoning over this wiki, treat all source code and
documentation as **upstream, read-only reference material**. No local
modifications to the code are expected or tracked.

- Do not propose edits to source files under `okf/`, `agents/`, `toolbox/`,
  or `samples/` — those are upstream.
- Wiki files under `Wiki/` are the only authored artifact in this fork.
- When summarizing capabilities, cite the actual source file rather than
  paraphrasing from memory.

---

## Source authority table

| Topic | Authoritative source |
|-------|----------------------|
| OKF specification | `okf/SPEC.md` |
| OKF enrichment agent (standalone) | `okf/README.md` |
| Full enrichment agent (Dataplex/ADK) | `agents/enrichment/README.md` |
| kcmd CLI reference | `agents/mdcode/README.md` |
| Repository overview | `README.md` |
| License | `LICENSE.md` |

---

## Bundle structure

```
Wiki/
├── CLAUDE.md              # This file — harness config
├── index.md               # Bundle root index
├── log.md                 # Update history
├── overview.md            # What the repo is and why it was forked
├── okf-spec.md            # OKF specification summary
└── components/
    ├── index.md           # Components directory listing
    ├── okf-agent.md       # Standalone OKF enrichment agent (okf/)
    ├── enrichment-agent.md # Full Dataplex enrichment agent (agents/enrichment/)
    └── kcmd.md            # Metadata as Code CLI (agents/mdcode/)
```

---

## OKF conformance note

Every concept document in this bundle carries a YAML frontmatter block with at
minimum `type`, `title`, `description`, and `updated`. The `source_files` field
lists the upstream files that the document summarizes. `index.md` and `log.md`
are reserved filenames per the OKF spec and carry no frontmatter.
