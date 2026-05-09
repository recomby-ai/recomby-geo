# recomby-geo

A Claude Code plugin that turns GEO (Generative Engine Optimization) from a
human-hour agency service into a reproducible 7-skill workflow with JSON
schema contracts between stages.

> **Status**: v0.1.0 — alpha. Schemas locked, skills v1, awaiting first
> end-to-end run on a real client.

---

## What it does

Given a client's raw materials (PDF / DOCX / URLs / notes), recomby-geo
walks the GEO loop end-to-end:

1. **Build a structured brand context** — 3-layer profile (identity / market
   reality / value gaps) aligned with Forrester ICP and Schema.org
   conventions.
2. **Audit visibility** across ChatGPT / Claude / Perplexity / Gemini for
   30+ target queries, with WebFetch-verified citations.
3. **Identify content priorities** using a CITE × CORE-EEAT scoring rubric.
4. **Generate briefs** with explicit human-in-loop slots — required real
   data, real cases, real expert quotes that AI can't auto-fill.
5. **Produce drafts** by applying Princeton KDD 2024 GEO rewrite techniques
   on the expert-filled brief.
6. **Distribute** with JSON-LD schema, internal-link plans, llms.txt
   fragments, third-party seeding suggestions.
7. **Re-audit monthly** and attribute movement to specific shipped actions
   with high / medium / low confidence levels.

Each skill writes a JSON file the next skill reads. The orchestrator is a
directory convention, not a state machine — you run skills in numerical
order, one client per folder.

---

## Why it exists

GEO agencies in 2026 charge monthly retainers for work that's now 80%
commoditized — there are 10+ mature open-source GEO/AEO Claude skills
([gego](https://github.com/AI2HU/gego),
[geo-aeo-tracker](https://github.com/danishashko/geo-aeo-tracker),
[aaron-he-zhu](https://github.com/aaron-he-zhu/seo-geo-claude-skills),
[Auriti-Labs](https://github.com/Auriti-Labs/geo-optimizer-skill), etc.)
that each solve a single step well.

The remaining 20% — **picking the right priorities given a brand's
differentiation, getting real expert input into briefs, and proving
attribution** — is what nobody has packaged. recomby-geo encodes that 20%
as orchestration glue and shims out the 80% to existing OSS.

---

## Install

This is a Claude Code marketplace plugin.

**Production install** (from a Claude Code session):
```
/plugin marketplace add recomby-ai/recomby-geo
/plugin install recomby-geo
```

**Local development**:
```bash
git clone https://github.com/recomby-ai/recomby-geo.git
# Then in Claude Code:
/plugin marketplace add /absolute/path/to/recomby-geo
/plugin install recomby-geo
```

### Required upstream skills

recomby-geo delegates to several skills that must be installed separately
in the consuming environment:

| Used by | Upstream |
|---------|----------|
| 05-production | `toprank:content-writer` |
| 06-distribution | `toprank:schema-markup-generator` |
| 04-content-brief | `toprank:meta-tags-optimizer` |
| 03-gap | `toprank:keyword-research` |
| 02-audit, 07-reaudit | one of: [`AI2HU/gego`](https://github.com/AI2HU/gego) or [`danishashko/geo-aeo-tracker`](https://github.com/danishashko/geo-aeo-tracker) |

---

## Quickstart

```bash
# 1. Make a client folder somewhere in your project (NOT inside this repo)
mkdir -p clients/acme/inputs

# 2. Drop materials into clients/acme/inputs/ (PDFs, decks, URLs, notes)

# 3. From Claude Code, with the plugin installed, run skills in order:
/01-intake          clients/acme
/02-audit           clients/acme
/03-gap             clients/acme
/04-content-brief   clients/acme --priority <id>
# (the brief now has REQUIRED-FILL slots — hand to your domain expert)
# (when expert returns it filled, re-run 04-content-brief Step 9 to verify)
/05-production      clients/acme --priority <id>
/06-distribution    clients/acme --priority <id>
# (publishing team uploads to CMS; record-publish appends to log.jsonl)

# Monthly:
/07-reaudit         clients/acme
```

Detailed orchestration, including the per-client folder layout, is in
[`plugins/recomby-geo/orchestrator/run.md`](plugins/recomby-geo/orchestrator/run.md).

---

## Skills

| # | Skill | One-liner |
|---|-------|-----------|
| 01 | `01-intake` | Ingest mixed-format materials → `brand_context.json` (3-layer Forrester ICP–aligned profile) |
| 02 | `02-audit` | Cross-LLM × cross-query mention/position/citation baseline → `visibility_baseline.json` (0–100 score) |
| 03 | `03-gap` | CITE × CORE-EEAT scored content priorities → `content_priorities.json` |
| 04 | `04-content-brief` | Brief with REQUIRED-FILL slots for domain expert → `briefs/<id>.md` (the human-in-loop checkpoint) |
| 05 | `05-production` | Apply Princeton KDD 2024 rewrite passes to filled brief → `drafts/<id>.md` |
| 06 | `06-distribution` | JSON-LD + internal links + llms.txt + 3rd-party targets → `distribution/<id>.json` |
| 07 | `07-reaudit` | Diff baselines + attribute movement to actions → `reaudit/round-N.json` |

Each skill's full spec is in `plugins/recomby-geo/skills/<NN-name>/SKILL.md`.

---

## Schemas

Four JSON Schemas (Draft 2020-12) under `plugins/recomby-geo/schemas/`:

| Schema | Purpose |
|--------|---------|
| `brand_context.schema.json` | 3-layer business context (identity / market reality / value gaps) + target queries + voice samples + content assets |
| `visibility_baseline.schema.json` | Cross-LLM mention/position/citation runs with 0–100 visibility score, compatible with gego / geo-aeo-tracker outputs |
| `content_priorities.schema.json` | Ranked priorities with `opportunity_type`, `recommended_format`, `required_assets` |
| `attribution_diff.schema.json` | Per-query movement attributed to specific shipped actions with confidence levels |

Validate any client output any time:

```bash
python3 -c "import json,jsonschema; \
  s=json.load(open('plugins/recomby-geo/schemas/brand_context.schema.json')); \
  d=json.load(open('clients/<slug>/brand_context.json')); \
  jsonschema.validate(d,s); print('OK')"
```

The schemas have teeth — 9 fixture tests covering accept-valid / reject-
invalid cases pass on every commit.

---

## Architecture in one diagram

```
inputs/  →  01-intake  →  brand_context.json
                            │
                            ▼
                         02-audit  →  visibility_baseline.json
                            │            │
                            └────┬───────┘
                                 ▼
                              03-gap  →  content_priorities.json
                                 │
                                 ▼
                       04-content-brief  →  briefs/<id>.md (with slots)
                                 │
                          [EXPERT FILLS]
                                 │
                                 ▼
                          05-production  →  drafts/<id>.md
                                 │
                                 ▼
                         06-distribution  →  distribution/<id>.json
                                 │
                          [PUBLISH + WAIT]
                                 │
                                 ▼ (monthly)
                            07-reaudit  →  reaudit/round-N.json
                                 │
                                 └──→ feeds next 03-gap run
```

---

## Built on

Skills shim out commodity work to mature open-source skills, then add the
orchestration / schema / human-in-loop layer no public skill ships:

- [`AI2HU/gego`](https://github.com/AI2HU/gego) — cross-LLM mention tracker (02-audit, 07-reaudit)
- [`danishashko/geo-aeo-tracker`](https://github.com/danishashko/geo-aeo-tracker) — alternative tracker, source of the 0–100 score convention
- [`aaron-he-zhu/seo-geo-claude-skills`](https://github.com/aaron-he-zhu/seo-geo-claude-skills) — CITE × CORE-EEAT framework (03-gap)
- [`Auriti-Labs/geo-optimizer-skill`](https://github.com/Auriti-Labs/geo-optimizer-skill) — Princeton KDD 2024 rewrite techniques (05-production), llms.txt convention (06-distribution)
- [`199-biotechnologies/claude-skill-seo-geo-optimizer`](https://github.com/199-biotechnologies/claude-skill-seo-geo-optimizer) — entity extraction patterns (06-distribution)
- [`GEO-optim/GEO`](https://github.com/GEO-optim/GEO) — Princeton KDD 2024 paper repo (empirical baseline for 03-gap, 05-production, 07-reaudit)
- Schema.org `Organization` / `Brand` + Forrester ICP — `brand_context` Layer 1 conventions

Reuse rule: **don't rebuild commodity, shim it; rebuild only the
differentiation (orchestration, schema contracts, human-in-loop gates,
attribution diff)**.

---

## Repo layout

```
recomby-geo/
├── .claude-plugin/marketplace.json
├── plugins/recomby-geo/
│   ├── plugin.json
│   ├── skills/{01-intake,02-audit,03-gap,04-content-brief,05-production,06-distribution,07-reaudit}/SKILL.md
│   ├── schemas/{brand_context,visibility_baseline,content_priorities,attribution_diff}.schema.json
│   └── orchestrator/run.md
├── README.md
└── .gitignore
```

Client folders (`clients/<slug>/`) live in the consumer's project, not
inside this plugin repo. They are gitignored by default.

---

## Status & roadmap

**v0.1.0 (current)** — Schemas locked, all 7 SKILL.md written, 9 schema
fixture tests passing.

**v0.2.0 (planned)** — Actual code shims around `gego` /
`geo-aeo-tracker` (Step 1 of 02-audit currently invokes via shell;
proper sub-module wiring is pending). First end-to-end real-client run.

**Out of scope for now** — multi-tenant orchestration, vertical-specific
schema branches (waiting for client #2 to motivate them), public-
marketplace license attributions (`THIRD_PARTY_LICENSES.md` pending).

---

## License

Private alpha. License will be added before any public release.
