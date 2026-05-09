# recomby-geo

GEO (Generative Engine Optimization) skill orchestration plugin for Claude
Code. Seven skills + four JSON schemas + a directory convention. Run it on
a per-client folder; ship content that AI engines actually cite.

> **Status**: v0.1.0 — alpha. Schemas locked; skills v1; awaiting first
> end-to-end run on `clients/sea-cicsic/`.

---

## What this is (and isn't)

**Is**: A workflow encoded as 7 chained skills. Each skill reads the
previous skill's structured output, does focused work, writes its own
structured output. The product is the workflow design + the JSON schemas
+ the human-in-loop checkpoint pattern.

**Isn't**: A new GEO content-writing AI. The community already has 10+
mature open-source GEO/AEO Claude skills. We *reuse* them as shims and add
the orchestration glue that nobody else has built: per-client folder
convention, JSON schema contracts between stages, and the expert-fills-
briefs human-in-loop gate that prevents the whole thing from collapsing
into AI-content-mill output.

---

## Why this exists

GEO agencies in 2026 charge monthly retainers for work that's now 80%
commoditized — mention auditing, cross-LLM prompting, schema markup,
"AI-friendly" content writing. Buying time, not a product.

The remaining 20% is hard:
- Picking the right priorities given a brand's actual differentiation.
- Building briefs that force the expert / founder to put real cases,
  real data, real opinions into the content (the things AI engines
  citing-rate at 3× higher than rephrased commodity content, per
  Princeton KDD 2024).
- Closing the loop with attribution: did this piece move that query?
  What didn't move and why?

That 20% is what recomby-geo encodes as skills. The 80% commodity work
is delegated to existing open-source skills via shims (see
`vendor-map.md`).

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

The orchestrator is a directory convention, not a state machine. See
`plugins/recomby-geo/orchestrator/run.md`.

---

## Install

This is a Claude Code plugin packaged as a marketplace. From inside Claude
Code:

```
/plugin add <this-repo-url>
```

Or install locally for development:

```
git clone <this-repo>
cd recomby-geo
# In Claude Code, add this directory as a marketplace.
```

### Dependencies (referenced, not bundled in v0.1.0)

The skills delegate to:
- `toprank:content-writer`, `toprank:schema-markup-generator`,
  `toprank:meta-tags-optimizer`, `toprank:keyword-research` (Recomby's
  toprank skill suite — install separately if not already present).
- `AI2HU/gego` OR `danishashko/geo-aeo-tracker` (one of the two — pick
  per `02-audit` Step 1 guidance).

See `vendor-map.md` for the full reuse map.

---

## Use

```bash
# 1. Make a client folder
mkdir -p clients/<slug>/inputs

# 2. Drop materials into clients/<slug>/inputs/

# 3. In Claude Code, with this plugin installed:
/01-intake clients/<slug>
/02-audit clients/<slug>
/03-gap clients/<slug>
/04-content-brief clients/<slug> --priority <id>
# (expert fills brief → run Step 9 of 04 to verify)
/05-production clients/<slug> --priority <id>
/06-distribution clients/<slug> --priority <id>
# (content gets published, log.jsonl appended)
/07-reaudit clients/<slug>      # monthly
```

Detailed orchestration in `plugins/recomby-geo/orchestrator/run.md`.

---

## Schemas

Four JSON schemas, all under `plugins/recomby-geo/schemas/`:

1. **`brand_context.schema.json`** — 3-layer business context (identity /
   market reality / value gaps) + target queries + voice samples + assets.
   Forrester ICP–aligned. Single source of truth for the pipeline.
2. **`visibility_baseline.schema.json`** — cross-LLM × cross-query
   measurement output. Aligned with `geo-aeo-tracker` 0-100 score
   convention.
3. **`content_priorities.schema.json`** — ranked priorities with
   `opportunity_type`, `recommended_format`, `required_assets`.
4. **`attribution_diff.schema.json`** — per-query movement attributed to
   specific shipped actions with confidence levels.

Validate any client's outputs at any time:

```bash
python3 -c "import json,jsonschema; \
  s=json.load(open('plugins/recomby-geo/schemas/brand_context.schema.json')); \
  d=json.load(open('clients/<slug>/brand_context.json')); \
  jsonschema.validate(d,s); print('OK')"
```

---

## What's not in v0.1.0

Deliberately out of scope:
- Multi-tenant orchestration with state machines / checkpoint gating.
  YAGNI for alpha. Directory convention is the orchestration.
- Actual code shims around `gego` / `geo-aeo-tracker` (skills currently
  invoke them via shell — wiring as proper sub-modules comes in 0.2).
- Vertical-specific schema branches (waiting for client #2 to learn
  whether we need them).
- Public-marketplace-ready license attributions
  (`THIRD_PARTY_LICENSES.md` is empty — populated before any public
  release).

---

## Repo layout

```
recomby-geo/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/recomby-geo/
│   ├── plugin.json
│   ├── skills/
│   │   ├── 01-intake/SKILL.md
│   │   ├── 02-audit/SKILL.md
│   │   ├── 03-gap/SKILL.md
│   │   ├── 04-content-brief/SKILL.md
│   │   ├── 05-production/SKILL.md
│   │   ├── 06-distribution/SKILL.md
│   │   └── 07-reaudit/SKILL.md
│   ├── schemas/
│   │   ├── brand_context.schema.json
│   │   ├── visibility_baseline.schema.json
│   │   ├── content_priorities.schema.json
│   │   └── attribution_diff.schema.json
│   └── orchestrator/run.md
├── vendor-map.md
├── README.md (this file)
└── .gitignore
```

Client folders (`clients/<slug>/`) live in the consumer's project, not
inside this plugin repo.
