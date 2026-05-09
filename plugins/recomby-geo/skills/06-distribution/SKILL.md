---
name: 06-distribution
description: >
  For a published-ready draft, generate JSON-LD schema markup, internal
  linking suggestions, third-party distribution targets, and llms.txt
  block. Outputs distribution/<id>.json and distribution/<id>.publish-bundle.md.
  Use after 05-production; consumed by 07-reaudit (records actions for
  attribution).
---

# 06 · Distribution & Schema — Publish-Ready Bundle

The skill that turns a draft into a deployable artifact: structured data
for SERP rich results AND for AI-engine entity recognition, internal
links to anchor topical authority, third-party seeding to build
cross-domain mention density.

---

## Inputs

- `clients/<slug>/brand_context.json`
- `clients/<slug>/content_priorities.json` (priority record for this id)
- `clients/<slug>/drafts/<id>.md`
- `clients/<slug>/drafts/<id>.meta.json`
- `clients/<slug>/content_assets[]` from brand_context (for internal-link
  candidates)

## Output

- `clients/<slug>/distribution/<id>.json` — machine-readable bundle:
  schema-jsonld, internal-link plan, external-link targets, llms.txt
  fragment.
- `clients/<slug>/distribution/<id>.publish-bundle.md` — human-readable
  publishing checklist for whoever uploads to the CMS.
- Append to `clients/<slug>/distribution/log.jsonl` — one line per
  publish event, used by 07-reaudit for attribution.

---

## Procedure

### Step 1 — Determine schema type

From the priority's `recommended_format`:

| format | Schema.org type |
|--------|-----------------|
| comparison-page | `Article` + `ItemList` of compared entities |
| deep-guide | `Article` + `HowTo` (if step-shaped) |
| case-study | `Article` + `Review` (with subject) |
| data-report | `Dataset` + `Article` |
| how-to | `HowTo` |
| faq-block | `FAQPage` |
| definition-page | `DefinedTerm` + `Article` |
| expert-essay | `Article` + `Person` (author) |
| list-roundup | `ItemList` + `Article` |

For all types, also include `Organization` (from brand_context) and
`BreadcrumbList`.

### Step 2 — Generate JSON-LD

Use `toprank:schema-markup-generator` as the actual generator (don't
re-implement). Pass:
- Article fields: headline, datePublished, author (from brand_context),
  publisher (Organization), articleBody (extracted from draft).
- Type-specific fields per Step 1 mapping.
- Author Person object with E-E-A-T signals (sameAs links, knowsAbout).

Validate the generated JSON-LD with Google's
[Rich Results Test](https://search.google.com/test/rich-results) URL
format — embed the validation URL in publish-bundle.md, don't auto-call.

### Step 3 — Internal linking plan

For each `content_assets[]` entry in brand_context:
- Compute topical relevance to current draft (keyword overlap of titles
  and `covers_query_ids`).
- Top 3–5 most relevant become internal-link candidates.

For each candidate:
- Suggest anchor text (use the target asset's primary query as anchor,
  not generic "click here").
- Suggest position in the draft (intro, body, conclusion).
- Mark as REQUIRED if the target asset is a category cornerstone, OPTIONAL
  otherwise.

Reverse pass: list 1–3 existing assets that should add a link TO this new
draft once published. The publish-bundle includes a "back-link adds"
checklist.

### Step 4 — External / third-party distribution targets

Pull from the priority's audience research (Layer 2
`real_user_questions[].source`). For each unique source:

- If Reddit / forum: add to `external_targets[]` with note "post a
  good-faith comment that links the new piece, ONLY if the thread is
  genuinely relevant. Spam = ban."
- If competitor blog comments / industry roundups: add to outreach list.
- If Wikipedia / Wikidata adjacent: add as suggested entity-graph edit
  (not a backlink, but improves AI's knowledge graph).

**Hard rule**: this list is suggestion, not automation. The user (Recomby
team or client) executes by hand. Auto-posting backlinks is a fast track
to penalty + ban.

### Step 5 — llms.txt fragment

Generate a llms.txt entry for this content (per emerging convention,
follows `Auriti-Labs/geo-optimizer-skill` recommendation):

```
- [<title>](<url>): <one-line summary tuned for AI parsing — answer-first,
  no marketing fluff>
```

The fragment goes into the bundle. The user appends to their site's
top-level `/llms.txt`.

### Step 6 — Write distribution.json

Schema:

```json
{
  "priority_id": "...",
  "draft_id": "...",
  "generated_at": "...",
  "schema_jsonld": { ... full JSON-LD ... },
  "internal_link_plan": [
    { "anchor": "...", "target_url": "...", "position": "intro|body|conclusion", "required": true }
  ],
  "back_link_adds": [
    { "from_url": "...", "to_anchor": "...", "rationale": "..." }
  ],
  "external_targets": [
    { "type": "reddit|forum|wikipedia|industry-blog", "url": "...", "action": "...", "tone": "..." }
  ],
  "llms_txt_fragment": "...",
  "publish_url_planned": null,
  "published_at": null,
  "published_url": null
}
```

`publish_url_planned` filled from CMS conventions (if `extended.website_backend.cms` is set in brand_context).

### Step 7 — Generate publish-bundle.md

Human-readable checklist for whoever pushes to the CMS:

```markdown
# Publish Bundle — <title>

## Pre-publish checks
- [ ] Run draft through Hemingway / Grammarly
- [ ] All slot annotations (<!-- slot: ... -->) removed before publishing
- [ ] All citation anchors resolve

## CMS upload
- [ ] Create page at <planned_url>
- [ ] Set publish date <ISO>
- [ ] Add meta title / description (auto-filled from draft frontmatter)
- [ ] Inline JSON-LD into <head>

## Post-publish
- [ ] Run page through https://search.google.com/test/rich-results
- [ ] Confirm Schema.org passes
- [ ] Submit URL via Google Search Console
- [ ] Add internal links per plan
- [ ] Add back-links from existing assets per plan
- [ ] Update top-level /llms.txt with fragment
- [ ] Reply to relevant external targets per outreach list

## Record back into recomby-geo
Once live, run: `recomby-geo record-publish --client <slug> --priority <id>
--url <published_url>` to append to distribution/log.jsonl. 07-reaudit
needs this for attribution.
```

### Step 8 — Append to distribution log

When the user later confirms publication, append one line to
`clients/<slug>/distribution/log.jsonl`:

```json
{"action_type":"content-published","action_id":"<priority-id>","title":"...","url":"...","shipped_at":"...","targets_query_ids":["..."]}
```

For schema deploy events, internal-link adds, external mentions —
separate lines with appropriate `action_type`.

---

## Hard Rules

1. **Validate JSON-LD before bundle ships** — reference the Rich Results
   Test URL.
2. **Strip slot annotations** — explicit step in the checklist; if any
   `<!-- slot: -->` survives to live page, AI engines may flag as spam.
3. **External outreach is human-executed** — never auto-post anywhere.
4. **Log every action** — distribution/log.jsonl is the input to
   07-reaudit attribution. Skip a log line and the diff can't credit the
   action.

---

## Reference

- `toprank:schema-markup-generator` — used directly for JSON-LD gen.
- `Auriti-Labs/geo-optimizer-skill` — llms.txt convention adoption.
- `199-biotechnologies/claude-skill-seo-geo-optimizer` — entity-graph
  edit ideas (Step 4 third-party).
- `geotoolco/Answer-Engine-Optimization` directory — finding "10+
  independent domains → 3.2× AI mention rate" empirical basis for
  external_targets discipline.

---

## Hand-off

Signal: *"Bundle ready for <id>. Hand to publishing team. Once live,
record-publish to log. Re-run 07-reaudit at month-end."*
