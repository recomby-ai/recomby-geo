---
name: 02-audit
description: >
  Run cross-LLM × cross-query mention/position/citation audit. For each query
  in brand_context.target_queries, send to multiple LLMs (ChatGPT, Claude,
  Perplexity, Gemini), record whether the client brand is mentioned, at what
  position, with what description, and which sources are cited. Outputs
  visibility_baseline.json validated against schema. This is the BASELINE —
  the reference point for 07-reaudit. Use after 01-intake completes; rerun
  monthly for re-audit rounds.
---

# 02 · Audit — Visibility Baseline Across LLMs

**This skill is a thin shim.** The community has solved this problem;
re-implementing wastes time and produces worse data. We delegate to one of
two open-source tools and conform their output to our schema.

---

## Inputs

- `clients/<slug>/brand_context.json` — required. Reads `target_queries`,
  `layer_1_business_identity.company.name`, `layer_1_business_identity.competitors`.

## Output

- `clients/<slug>/visibility_baseline.json` (round 1) — validates against
  `schemas/visibility_baseline.schema.json`.
- `clients/<slug>/baselines/round-N.json` (round 2+) — preserved snapshots
  for 07-reaudit diff.
- `clients/<slug>/baseline-report.md` — human-readable summary for the
  monthly review meeting.

---

## Procedure

### Step 1 — Pick the audit tool

Use **`AI2HU/gego`** by default. It's small, local-first, supports
ChatGPT/Claude/Perplexity/Gemini, and outputs JSON we can normalize.

Fallback: **`danishashko/geo-aeo-tracker`** if gego doesn't support a
required model. Has a richer dashboard but heavier dependency footprint.

Last resort: native implementation using direct API calls to Anthropic,
OpenAI, Perplexity, Gemini SDKs. Only if both shims fail.

Tool choice is recorded in `meta.tool_used`.

### Step 2 — Prepare query list

```jq
jq '.target_queries | map({query, query_id: .query | gsub(" "; "-") | ascii_downcase})' \
  clients/<slug>/brand_context.json
```

Strip any P2 queries if the client is on a tight budget; default is to run
all P0 + P1 + P2.

### Step 3 — Run the audit

Invoke the chosen tool. For gego, typical invocation:

```bash
gego run \
  --queries-file <queries.json> \
  --models chatgpt,claude,perplexity,gemini \
  --brand "<company.name>" \
  --competitors "<comp1,comp2,comp3>" \
  --output raw-baseline.json
```

Each (query × model) pair = one row. For 30 queries × 4 models = 120 rows.

### Step 4 — Normalize tool output to our schema

Map raw tool output to `schemas/visibility_baseline.schema.json` shape.
Fields the tool may not provide directly — derive them:

- `mentioned`: regex search of brand name + known aliases in `raw_response`.
- `position`: parse from list-format responses; `null` if not mentioned.
- `description_quoted`: extract the sentence containing the brand name.
- `competitors_mentioned`: regex from competitor list.
- `is_owned_by_client`: domain match against `company.url`.

### Step 5 — Verify citations live

For each unique URL in `runs[*].citations[*]`, `WebFetch` to confirm it
resolves and is on-topic. Set `verified_live` accordingly. Skip URLs
already verified in this client's previous baseline (cache).

This is critical — AI hallucinated citations are common, and ruining the
baseline with hallucinated sources breaks 07-reaudit attribution.

### Step 6 — Compute summary

```python
mention_rate = sum(r.mentioned for r in runs) / len(runs)
positions = [r.position for r in runs if r.mentioned and r.position]
avg_position_when_mentioned = sum(positions)/len(positions) if positions else None
owned_citation_rate = sum(c.is_owned_by_client for r in runs for c in r.citations) / max(1, total_citations)
overall_visibility_score = round(
  100 * (0.5*mention_rate + 0.3*(1/(avg_position_when_mentioned or 10)) + 0.2*owned_citation_rate),
  1
)
```

`overall_visibility_score` is 0–100, aligned with `geo-aeo-tracker` convention.

Per-query verdict logic:
- `mention_rate >= 0.6` and `avg_position <= 3` → `winning`
- `0 < mention_rate < 0.6` → `contested`
- `mention_rate == 0` → `absent`
- (only in re-audit rounds) `mention_rate < previous - 0.1` → `regressing`

### Step 7 — Write outputs

- Round 1: `clients/<slug>/visibility_baseline.json`
- Round N≥2: `clients/<slug>/baselines/round-N.json` AND replace
  `visibility_baseline.json` with the new round.

Generate `baseline-report.md`:

```markdown
# Baseline Report — <client> — round <N>
> captured: <ISO ts>
> tool: <tool_used>

## Headline
- Overall visibility: **<score>/100**
- Mention rate: <pct>%
- Avg position when mentioned: <num>
- Owned citation rate: <pct>%

## Top wins
<table of queries with verdict=winning>

## Top gaps
<table of queries with verdict=absent and high-priority>

## Competitor landscape
<table>
```

### Step 8 — Validate

```bash
python3 -c "import json,jsonschema; \
  s=json.load(open('plugins/recomby-geo/schemas/visibility_baseline.schema.json')); \
  d=json.load(open('clients/<slug>/visibility_baseline.json')); \
  jsonschema.validate(d,s); print('OK')"
```

---

## Hard Rules

1. **Schema-validated output**.
2. **No fabricated runs** — if a model API errors out, skip and document; do
   not synthesize a fake response. The baseline is a measurement, not a guess.
3. **Citation freshness** — every URL in `citations[]` must have
   `verified_live` set. Don't let hallucinated URLs into the diff dataset.
4. **Preserve raw_response** — even if it's long. Required for 07-reaudit
   to do real diffs against next round.
5. **Round numbering** — `meta.audit_round` is 1 the first time, increments
   only when 07-reaudit calls back into 02-audit.

---

## Reference

- `AI2HU/gego` — primary shim target.
- `danishashko/geo-aeo-tracker` — fallback shim. 0–100 visibility score
  convention adopted from this tool.
- `Auriti-Labs/geo-optimizer-skill` — Princeton KDD 2024 GEO methodology;
  consult for what makes a citation "high authority" in the verify step.

---

## Hand-off

Signal: *"Baseline captured. round=<N>, score=<X>/100. Run 03-gap next."*
