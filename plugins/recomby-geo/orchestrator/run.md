# Orchestrator — How to Run recomby-geo

There is no state machine. The orchestration is a directory convention plus
this document. Read this once; then it's just running skills in order.

---

## Per-client folder layout

Every client gets one folder under `clients/<slug>/` (slug is lowercase
kebab, e.g., `sea-cicsic`, `acme-saas`). Contents:

```
clients/<slug>/
├── inputs/                      # raw materials from the client (PDF/DOCX/URLs/notes)
├── brand_context.json           # ← 01-intake writes
├── intake-log.md                # ← 01-intake appends
├── visibility_baseline.json     # ← 02-audit writes (current)
├── baselines/                   # historical baselines preserved per round
│   └── round-N.json
├── baseline-report.md           # ← 02-audit writes
├── content_priorities.json      # ← 03-gap writes
├── briefs/
│   ├── <priority-id>.md         # ← 04-content-brief writes
│   └── <priority-id>.meta.json
├── drafts/
│   ├── <priority-id>.md         # ← 05-production writes
│   └── <priority-id>.meta.json
├── distribution/
│   ├── <priority-id>.json       # ← 06-distribution writes
│   ├── <priority-id>.publish-bundle.md
│   └── log.jsonl                # append-only action log (publishes, schema deploys, etc.)
└── reaudit/
    ├── round-N.json             # ← 07-reaudit writes
    └── round-N.report.md
```

---

## First-time run for a new client

1. Create `clients/<slug>/inputs/`. Drop in everything the client gave
   you: PDFs, decks, URL list, voice samples, raw founder notes.
2. Run **01-intake**. Outputs `brand_context.json`. Stop and confirm
   accuracy with the client before proceeding.
3. Run **02-audit**. Outputs `visibility_baseline.json` (round 1) and
   `baseline-report.md`. Send report to client.
4. Run **03-gap**. Outputs `content_priorities.json`. Review with client
   before committing to the top 3.
5. For each top-3 priority:
   - Run **04-content-brief** with `priority_id`. Outputs `briefs/<id>.md`
     with REQUIRED-FILL slots.
   - **Hand brief to expert / founder / domain specialist.** They fill the
     slots — this is the human-in-loop checkpoint that makes the whole
     thing GEO-defensible.
   - When expert returns the filled brief, re-run **04-content-brief
     Step 9** to verify and flip status to `ready-for-production`.
   - Run **05-production**. Outputs `drafts/<id>.md`.
   - Run **06-distribution**. Outputs `distribution/<id>.json` and the
     publish-bundle checklist.
   - Hand publish-bundle to whoever uploads to the CMS.
   - When live, run `record-publish` (see below) to append to
     `distribution/log.jsonl`.

## Monthly cycle

End of each month:
1. Run **07-reaudit**. It internally calls 02-audit again, then diffs.
   Outputs `reaudit/round-<N>.json` and the report.
2. Use the report's `next_round_recommendations` as input to the next
   03-gap run (which produces a refreshed `content_priorities.json`).
3. Repeat: pick top-N from refreshed priorities → 04 → 05 → 06.

---

## Recording a publish event

When content goes live, append a line to
`clients/<slug>/distribution/log.jsonl`:

```json
{"action_type":"content-published","action_id":"<priority-id>","title":"<title>","url":"<live url>","shipped_at":"<ISO ts>","targets_query_ids":["<id1>","<id2>"]}
```

For schema deploys, internal-link adds, third-party mentions — append
similarly with appropriate `action_type`. 07-reaudit reads this file to
attribute movement.

---

## Hard rules at orchestration level

1. **Never run a skill out of order.** 02 needs 01's output; 03 needs
   01+02; 04 needs 01+03; 05 needs 04 (filled); 06 needs 05; 07 needs
   prior 02.
2. **Schema validation is enforced** by each skill. If validation fails,
   fix before moving on.
3. **One client per folder.** Don't share JSON across folders. Don't try
   to "factor out" common context — that's the agency-mind talking.
4. **The expert fills briefs.** Not Claude. If the expert isn't
   available, pause the pipeline. Don't auto-fill.

---

## Skill dependency graph

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
                                                 + publish-bundle.md
                                                 + log.jsonl entry
                                 │
                          [PUBLISH + WAIT]
                                 │
                                 ▼ (monthly)
                            07-reaudit  →  reaudit/round-N.json
                                 │
                                 └──→ feeds next 03-gap run
```
