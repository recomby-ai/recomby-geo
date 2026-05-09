# Vendor Map — What recomby-geo Borrows From the Community

Principle: don't rebuild commodity. Each skill below documents (a) the
upstream open-source repo we lean on, (b) what we add on top, (c) license
and sync notes.

---

## 01-intake

**Upstream**: none directly — Intake is mostly bespoke because no public
GEO skill cleanly handles the "ingest mixed-format raw materials → GEO-
schema'd brand context" flow with a hard schema gate.

**Inspirations**:
- `aaron-he-zhu/seo-geo-claude-skills` — query-discovery patterns.
- `geotoolco/Answer-Engine-Optimization` — directory of community-
  validated AEO concepts feeding our 5-core-field gate.
- Schema.org `Organization` / `Brand` — Layer 1 field naming.
- Forrester ICP / ChiefMartec persona — Layer 1 `target_customer`
  structure.

**Add-on (our differentiation)**:
- 3-layer schema (identity / market / gaps) with hard gates per layer.
- WebFetch-verify-before-trust rule on competitors and citations.
- Confirm-before-write protocol.

---

## 02-audit

**Upstream (primary)**: `AI2HU/gego` —
https://github.com/AI2HU/gego
- Cross-LLM prompt runner, keyword extraction, JSON output.
- License: check repo (MIT-likely; verify before bundling).

**Upstream (fallback)**: `danishashko/geo-aeo-tracker` —
https://github.com/danishashko/geo-aeo-tracker
- 6-model coverage, 0–100 visibility score (we adopted this convention).
- Heavier dep footprint; use if gego doesn't cover a needed model.

**Add-on**:
- Schema normalization to our `visibility_baseline.schema.json`.
- WebFetch-verify-live on every cited URL (catches AI hallucinated
  citations).
- Round numbering convention for cross-round diff in 07-reaudit.
- Per-query verdict assignment (`winning` / `contested` / `absent` /
  `regressing`).

**Sync strategy**: pin to a specific tag in the shim's invocation
documentation. Re-evaluate quarterly. Don't auto-track upstream HEAD —
audit data must be reproducible across rounds.

---

## 03-gap

**Upstream (framework borrow)**: `aaron-he-zhu/seo-geo-claude-skills` —
https://github.com/aaron-he-zhu/seo-geo-claude-skills
- CITE + CORE-EEAT scoring rubric.
- License: MIT (verify).

**Upstream (alternative scoring)**: `Bhanunamikaze/Agentic-SEO-Skill` —
https://github.com/Bhanunamikaze/Agentic-SEO-Skill
- 16 sub-skill scoring dimensions; consult for edge-case scoring.

**Empirical basis**: Princeton KDD 2024 (`GEO-optim/GEO` —
https://github.com/GEO-optim/GEO) — which content moves AI citations.

**Add-on**:
- `opportunity_type` taxonomy (fill-absence / displace-competitor /
  deepen-existing / etc.).
- `required_assets` discipline — every priority must list at least one
  expert-only input. This is the moat against AI-content-mill drift.
- 5-per-type cap forcing diversification.

---

## 04-content-brief

**Upstream**: none — this is the core moat skill. No public skill
implements the human-in-loop slot discipline this way.

**Inspirations**:
- `toprank:content-writer` — outline scaffolding mechanics.
- `toprank:meta-tags-optimizer` — title/headline conventions.
- Princeton KDD 2024 — statistics + quotation + citation block
  empirical motivation.
- `199-biotechnologies/claude-skill-seo-geo-optimizer` — entity
  extraction for citation pre-population.

**Add-on**:
- REQUIRED-FILL slot syntax with id / type / what-goes-here / why-it-
  matters / bad-example / good-example.
- Status state machine: `awaiting-expert-fill` → `ready-for-production`,
  enforced as a hard gate by 05-production.
- "AI-fillability test" — every slot must require brand-specific
  knowledge.

---

## 05-production

**Upstream (delegation)**: `toprank:content-writer` (your own toprank
skill set) — outline-to-prose mechanics.

**Upstream (rewrite techniques)**: `Auriti-Labs/geo-optimizer-skill` —
https://github.com/Auriti-Labs/geo-optimizer-skill
- Princeton KDD 2024 rewrite passes (quotation injection, statistics
  surfacing, citation densification, authoritative phrasing, fluency).
- License: check repo.

**Add-on**:
- Hard refusal gate on unfilled briefs (Step 1).
- Slot annotation preservation (HTML comments) for downstream
  attribution.
- Voice-match scoring against `brand_context.voice_samples`.
- GEO readiness checklist (definition / FAQ / comparison / methodology /
  answer-first-200-words).

---

## 06-distribution

**Upstream (JSON-LD generation)**: `toprank:schema-markup-generator` —
direct invocation. Don't reimplement.

**Upstream (llms.txt convention)**: `Auriti-Labs/geo-optimizer-skill` —
their llms.txt format.

**Upstream (entity-graph ideas)**: `199-biotechnologies/claude-skill-
seo-geo-optimizer`.

**Empirical basis**: `geotoolco/Answer-Engine-Optimization` directory —
"10+ independent domains → 3.2× AI mention rate."

**Add-on**:
- Internal-link plan tied to brand_context.content_assets keyword
  overlap.
- Reverse back-link pass (existing assets → new piece).
- External-targets discipline: human-executed, never auto-posted.
- distribution/log.jsonl convention for 07-reaudit attribution.

---

## 07-reaudit

**Upstream (re-uses)**: 02-audit (which itself uses gego/geo-aeo-tracker).

**Empirical basis**: Princeton KDD 2024 — what content moves visibility
informs the `actions_without_impact.diagnosis` taxonomy.

**Add-on (this is the most differentiated skill)**:
- attribution_diff schema — per-query movement attributed to specific
  actions, with confidence levels (high / medium / low / unknown).
- Reverse-direction action audit (what shipped, did it move anything).
- Diagnosis taxonomy: too-recent / wrong-target / weak-content /
  distribution-gap / competitor-counter / unknown.
- 7-day re-index buffer rule (actions <7 days old auto-flagged
  too-recent, not weak).
- next_round_recommendations as machine-readable hand-off into next
  03-gap.

---

## Toprank skill dependencies (already installed in user's CC)

These are referenced as direct invocations, not bundled:
- `toprank:content-writer` — used by 05-production.
- `toprank:schema-markup-generator` — used by 06-distribution.
- `toprank:meta-tags-optimizer` — referenced by 04 / 06 for headlines.
- `toprank:keyword-research` — referenced by 03-gap for adjacent-query
  expansion.

If a downstream user doesn't have toprank installed, recomby-geo's
delegations will fail — install instructions in README.

---

## License watch

Before publishing this plugin (even privately) and before bundling any
upstream code:
- [ ] Verify license on `AI2HU/gego` (likely MIT; confirm).
- [ ] Verify license on `danishashko/geo-aeo-tracker`.
- [ ] Verify license on `aaron-he-zhu/seo-geo-claude-skills`.
- [ ] Verify license on `Auriti-Labs/geo-optimizer-skill`.
- [ ] Add THIRD_PARTY_LICENSES.md with attribution before any public release.

For now (private repo, alpha use), references are documentation only —
no upstream code is bundled. When we add actual shim code (subprocess
calls or vendored modules), license obligations attach.
