# PROJECT_BRIEF.md — Continuum (v2)

> **How to use this file**
> - **Google AI Studio:** paste into *System Instructions* for every build session.
> - **Google Antigravity:** save as `AGENTS.md` in the repo root.
> - This brief is the single source of truth for product intent, UX, tech stack and build rules.
> - Companion files: `DATA_SPEC.md`, `GOVERNANCE_CONFIG.json`, `/data/*.json` (pre-generated mock data), `SCREEN_SPEC.md`, `PROMPT_PACK.md`. If this brief and a companion file conflict, the companion file wins for its own domain (data, governance, screens).
>
> **Changes from v1:** LLM-agnostic AI layer (no Gemini/vendor SDK dependency) · persona-led navigation (5 groups, 14 screens) · full coverage of the reference-architecture service components (Change Monitor pipeline, signal events, thresholds, rights matrix, feature store, consent, mock CRM record, downstream consumers) · pre-generated JSON mock data instead of an in-app generator · new AI moment AI-5 (rep-note signal extraction).

---

## 1. Product summary

**Name:** Continuum
**Tagline:** *Governed, living segmentation across every market.*
**Category:** Enterprise commercial analytics application for pharma / life sciences.

**What it does.** Continuum lets a global commercial organisation run HCP and account (HCO) segmentation as **one governed process across markets with very different data maturity**. It:
1. **Standardises** fragmented market processes (in-house models, different tools, agency Excel files) into one global segmentation vocabulary.
2. **Builds** segments in a Segmentation Studio (rules or K-means), within market rights, and versions them in a library.
3. **Keeps segments current** between planning cycles: watches signals, detects drift, explains it, proposes changes, routes them to the named owner, and writes approved changes back to CRM.
4. **Governs and proves value**: segment health, audit trail, controlled learning, bias checks and a value calculator.

**Positioning line (show on login / About):**
*"Analyst workbenches build segments. Continuum runs segmentation as a governed global process — every market, every data reality, from studio to CRM."*

**Definition used throughout the app:**
Living segmentation is the governed capability to **detect, explain, approve and operationalise** material changes in customer segment attributes between formal planning cycles. It is **not** constant automated re-clustering.

**Nature of this build:** a demo application on **synthetic data**. It must look and behave like an enterprise-grade product. No real backend, no real CRM, no real customer data.

---

## 2. Non-negotiable product principles

These must be visible in the UI and enforced in logic.

1. **AI proposes, people decide.** No AI or model output ever changes a governed segment field directly. Every change is a *proposal* that a named owner approves or rejects.
2. **Nothing is written back unapproved.** CRM write-back happens only after approval. The audit log must show 0 unapproved write-backs.
3. **Every proposal carries evidence:** drivers, confidence (High / Medium / Low), data age, source signals.
4. **Segment age is visible wherever a segment is shown** (chip, e.g. "164 days").
5. **Global vocabulary, local control.** Global defines dimensions and allowed values; markets configure signals, thresholds, cadence and approval rights.
6. **Stability policy is enforced:** freeze windows, change-frequency limits, minimum evidence; conflicting signals → *Hold for review* (never auto-proposed).
7. **Learning is controlled:** approvals, rejections and downstream outcomes feed a *recalibration recommendation* that a model owner validates. The app never shows auto-retraining.
8. **No promotional language** in any AI-generated text. Explanations are factual and grounded in drivers only.
9. **Medical is firewalled.** No Medical/MSL personas or medical KOL segmentation in scope.
10. **Coexistence, not replacement.** Continuum publishes to CRM and downstream consumers (targeting, call planning, NBA, journeys); it replaces none of them.
11. **Model-agnostic AI.** Any LLM can be plugged in; the demo runs fully without one.

---

## 3. Scope

### In scope
| | HCP segmentation | Account (HCO) segmentation |
|---|---|---|
| Unit | Prescriber (specialist, GP, NP/PA) | Hospital, clinic, IDN/hospital group, specialty centre, pharmacy chain |
| Dimensions | Segment (A–E), Potential (1–4), Adoption stage, Behavioural/attitudinal segment, Channel preference / digital affinity, Micro-segment | Account archetype, Account potential, Access/formulary status, Decision-making model, Treatment capability, Account tier |
| Owner | Back office / commercial ops; reps own adoption stage and digital behaviour | KAM / account team / commercial excellence |
| Link | HCP–HCO affiliations | HCP potential rolls up to account potential; account access status caps HCP opportunity |

### Out of scope (do not build)
Payer segmentation, patient segmentation, Medical/KOL segmentation, territory alignment, NBA engine logic, real CRM integration, authentication.

---

## 4. Demo world

### 4.1 Markets
Labels come from `/data/markets.json`. Defaults:

| Code | Display name | Maturity | Profile | Current process | Studio method | Living signal | Full universe HCPs / Accounts |
|---|---|---|---|---|---|---|---|
| MKT_A | Market A · Data-rich | Data-rich | HCP-level Rx and claims, digital, KOL | In-house model, quarterly refresh | K-means → rules overlay | Near-continuous re-scoring | 8,000 / 900 |
| MKT_B | Market B · Signal-enriched | Signal-enriched | Brick-level sales, strong CRM, consented digital | Different tool, annual refresh, rep adjustments | Rules/deciling + behavioural clusters | Several moving signals | 4,000 / 400 |
| MKT_C | Market C · Survey-led | Survey-led / manual | No HCP-level sales; PMR + CRM + rep input | Local agency Excel file yearly + rep judgement | Global rules template (potential grid with weights) | Rep observation + CRM activity | 1,500 / 150 |

### 4.2 Brand portfolio (all fictitious)
| Brand | Therapy area | Indications | Lifecycle | Markets | Demo role |
|---|---|---|---|---|---|
| **Aurelix** | Immunology | Plaque psoriasis; Psoriatic arthritis | Launch | A, B, C | **Hero brand** — living loop |
| **Zentrova** | Oncology | NSCLC; HER2+ breast cancer | Growth | A, B, C | Account-led; formulary win → HCP propagation |
| **Cardivance** | Cardiometabolic | Heart failure; CKD | Mature | A, B, C | Stable segments; "not every signal changes the segment" |
| **Neurelle** | Neuroscience | Migraine prevention; Multiple sclerosis | Growth | A, B | Digital-heavy; channel affinity, cross-segmentation |
| **Brevanta** | Respiratory | Severe asthma; COPD | Mature | A, B | GP + specialist mix; bias check (urban/rural) |

Default selection on load: **Aurelix**, **All markets**.

### 4.3 Personas
| ID | Persona | Market scope | Can act |
|---|---|---|---|
| P1 | **Global Segmentation Lead** | All | Edit standards, Studio templates, approve/activate library versions, approve recalibration |
| P2 | **Market Back Office** (Commercial Excellence) | Own market (default MKT_B) | Run Studio within market rights, approve/reject HCP proposals, publish |
| P3 | **KAM / Account Lead** | Own market accounts + affiliated HCPs | Approve/reject account archetype, tier, access proposals |
| P4 | **Field Rep** | Own territory HCPs | Validate adoption stage and digital behaviour; reject with reason |
| P5 | **Local Agency** (vendor) | MKT_C intake only | Upload segmentation file, view validation results |
| P6 | **Executive / Compliance** (CCO, Head of Comm Ex, CDAO, Compliance) | All, read-only | None; export |

---

## 5. Information architecture

Five plain-language groups, 14 screens. Detail views (Customer 360) open from lists, not from the nav. **Ask Continuum** is a floating assistant available on every screen.

```
CONTINUUM
│
├─ OVERVIEW                       "Is segmentation healthy and worth it?"
│   ├─ S01  Executive Cockpit
│   └─ S02  Value Calculator
│
├─ STANDARDS & DATA               "What are the rules, and is our data ready?"
│   ├─ S03  Global Standards      tabs: Dimensions · Approval Rights · Thresholds · Stability Policy
│   ├─ S04  Market Data           tabs: Readiness · Data Quality · Signal Freshness · Feature Store
│   └─ S05  Data Intake
│
├─ SEGMENTS                       "What segments do we have, and what do they tell us?"
│   ├─ S06  Segmentation Studio   steps: Filter · Build · Review · Map to standard
│   ├─ S07  Segment Library       versions + status lifecycle + bulk refresh
│   └─ S08  Segment Insights      tabs: Profiles · Opportunity · Migration · Accounts ·
│                                       Cross-market · Cross-segmentation · Targeting alignment
│
├─ CHANGES                        "What changed, and what do we do about it?"
│   ├─ S09  Change Monitor        tabs: Pipeline · Signal Feed · Drift Flags
│   ├─ S10  Review Queue          → opens D01 Customer 360 (HCP / Account)
│   └─ S11  Publish to CRM        CRM record view · downstream consumers · publish log
│
├─ CONTROLS                       "Is it under control and fair?"
│   ├─ S12  Segment Health        tabs: Age · Pipeline KPIs · Time to Update · Field Trust · Standardisation
│   ├─ S13  Audit & Learning      tabs: Change Log · Version History · Recalibration
│   └─ S14  Responsible AI
│
└─ Ask Continuum  (floating panel, S15)
```

The **Changes** group follows the service order of the reference architecture (watch → detect → explain → propose → record), so the nav itself tells the story.

### 5.1 Persona navigation and landing pages
| Persona | Lands on | Visible nav | Label overrides |
|---|---|---|---|
| P1 Global Segmentation Lead | S03 Global Standards | All groups | — |
| P2 Market Back Office | S10 Review Queue | Segments · Changes · Standards & Data (S04 own market only) | — |
| P3 KAM / Account Lead | S10 Review Queue (filter: Accounts) | Changes (S10, S11) · S08 (Accounts tab) | — |
| P4 Field Rep | S10 shown as **"My Customers"** | S10 only (+ D01) | S10 → "My Customers" |
| P5 Local Agency | S05 Data Intake | S05 only | — |
| P6 Executive / Compliance | S01 Executive Cockpit | Overview · Controls · S08 (read-only) | — |

Persona switcher (top bar) changes landing page, visible nav, enabled actions and market scope. Disabled actions show a tooltip: *"Your role cannot approve this field in this market."*

---

## 6. Reference-architecture coverage

Every component of the living segmentation reference architecture must be visible in the app.

| Architecture component | AI intervention | Where it appears |
|---|---|---|
| **Data sources** (survey/PMR, CRM activity & notes, sales/Rx, claims & access, digital, reference & affiliations) | — | S04 Readiness heatmap (availability varies by market) |
| **Lakehouse / customer master** | — | S04 Data Quality (match rate, duplicates, affiliations) |
| **Consent & preference** | — | D01 Customer 360 (consent panel); S11 audience counts exclude non-consented channels |
| **Dimension catalogue** *(new)* | — | S03 Dimensions tab (values per brand · market · version) |
| **Feature store** *(new)* | — | S04 Feature Store tab (feature, source, refresh, data age, market availability) |
| **Maintenance-rights matrix** *(new)* | Rules | S03 Approval Rights tab |
| **1. Signal watch** → change events | **Agent** (scheduled scan) · **GenAI** (reads rep notes, AI-5) | S09 Signal Feed |
| **2. Drift detection** → drift flags + confidence | **ML** (classifier + confidence), market thresholds | S09 Drift Flags; thresholds in S03 |
| **3. Explanation** → explanation card | **GenAI** grounded in drivers (AI-3) | S10, D01 |
| **4. Proposal & routing** → proposed segment | **Rules** (rights matrix) · **Human** approves | S10 Review Queue |
| **5. Record & learn** → audit trail, recalibration proposal | **ML** monitored, not auto-retrained | S13 |
| **Guardrails** | — | Shown as a strip on S09 Pipeline and enforced in logic |
| **CRM** (segment field, proposed-segment field, approval workflow, segment age) | — | S11 mock CRM record view |
| **Downstream consumers** (targeting, call planning, NBA, journeys, segment-health dashboard) | — | S11 consumer tiles with affected counts |
| **Feedback loop** (approvals, rejections, reach, coverage of rising customers, overrides) | — | S13 Recalibration tab |
| **Who decides** (runs on its own · needs approval · stays human-owned) | — | Band at the bottom of S09 Pipeline |

### 6.1 S09 Change Monitor — Pipeline tab (signature screen)
A horizontal five-stage flow mirroring the architecture slide:

```
[1 Signal watch] → [2 Drift detection] → [3 Explanation] → [4 Proposal & routing] → [5 Record & learn] → CRM
   1,240 events       312 flags            312 cards          186 proposals            97 written back
   Agent · GenAI      ML                   GenAI              Rules · Human (gold)     ML
                                                              41 on hold
```
- Each stage card: name, one-line description, AI-intervention badges, today's count, output label (Change events → Drift flags + confidence → Explanation card → Proposed segment → Audit trail · recalibration proposal).
- Stage 4 is gold (the only human gate). Clicking a stage routes to its detail (Signal Feed, Drift Flags, Review Queue, Publish, Audit).
- Below: guardrails strip; feedback-loop arrow; "Who decides" band.
- Counts filter by market and brand and update live after approvals.

---

## 7. Hero storyline (must be flawless)

~25 minutes, 8 steps. Every screen used here must work with the hero records in `DATA_SPEC.md`.

| Step | Moment | Screens |
|---|---|---|
| 1 | The problem: three markets, three processes, segments that can't be compared | S01 → S12 (Standardisation) → S08 (Cross-market) |
| 2 | Why markets differ: data realities | S04 (Readiness, Data Quality) |
| 3 | Onboard Market C: vendor Excel → AI mapping → match rate → conformance | S05 |
| 4 | Studio: Market C runs rules template, Market A runs K-means; both map to the global standard and save as versions | S06 → S07 → S08 (Profiles) |
| 5 | Output insight: opportunity gaps and account landscape | S08 (Opportunity, Accounts, Cross-segmentation) |
| 6 | Living loop: formulary win at hero account (Zentrova, MKT_B) and a rep note trigger change events → drift → proposals; hero HCP (Aurelix) explained and approved | S09 (Pipeline → Signal Feed) → S10 → D01 |
| 7 | Publish: CRM write-back, call plan and audiences shift | S11 → S08 (Targeting alignment) |
| 8 | Govern & prove: health, audit, bias, value | S12 → S13 → S14 → S02 → S01 |

A **"Guided demo"** toggle in the top bar shows a step indicator (1–8) with *Next*, which routes to the right screen with filters and persona pre-set.

---

## 8. Tech stack and project structure

**Stack:** React 18 + TypeScript + Vite · Tailwind CSS · Recharts · `d3-sankey` · React Router · Zustand · Zod (schema validation) · SheetJS `xlsx` (vendor file) · lucide-react (icons). **No LLM vendor SDKs** (no `@google/genai`, `openai`, `@anthropic-ai/sdk`, etc.).

```
/data                       pre-generated mock data (see Section 9)
/prompts                    provider-neutral prompt templates (AI-1 … AI-5)
/server/llm-proxy           OPTIONAL ~50-line proxy holding the API key (live mode only)
/src
  /config        appConfig.ts      (app name, theme, demo date, feature flags)
  /llm           LLMProvider.ts    (interface)
                 providers/MockProvider.ts
                 providers/OpenAICompatibleProvider.ts
                 providers/AnthropicProvider.ts
                 providers/GeminiProvider.ts   (optional)
                 llmConfig.ts      (provider, model, baseUrl, temperature, timeoutMs)
                 schemas.ts        (Zod schema per AI task)
  /services      dataService.ts        (ONLY way screens read data)
                 aiService.ts          (ONLY way screens call AI)
                 governanceService.ts  (rights, thresholds, stability policy checks)
  /store         useAppStore.ts    (persona, market, brand, guided step, proposals,
                                    approvals, library, audit log, publish log)
  /components    layout/  AppShell, SideNav, TopBar, PageHeader
                 ui/      KpiTile, DataTable, StatusBadge, ConfidenceChip, SegmentAgeChip,
                          InterventionBadge, ApproverPill, FilterBar, Tabs, Drawer, Modal,
                          EmptyState, Toast, StepIndicator, AIBadge
                 charts/  Heatmap, Funnel, Sankey, BoxPlot, Matrix4x4, Bar, Line, Donut,
                          PipelineFlow
  /screens       S01_ExecutiveCockpit.tsx … S14_ResponsibleAI.tsx,
                 D01_Customer360.tsx, S15_AskContinuum.tsx
  types.ts, App.tsx, main.tsx, routes.tsx
```

---

## 9. Data approach

Mock data is **pre-generated and supplied as JSON files** in `/data`. The app does not generate data.

### 9.1 Rules
1. Screens **never** hard-code data. All reads go through `dataService.ts`. Later, only `dataService.ts` changes to call an API/database.
2. Files are **table-shaped**: one entity per file, flat records, explicit keys (`hcp_id`, `hco_id`, `market_code`, `brand_code`). Each file maps 1:1 to a future database table.
3. **Row-level sample + aggregates.** Row files hold a representative sample (~600 / 400 / 250 HCPs; ~120 / 80 / 40 accounts for A / B / C). Full-universe totals and distributions live in `aggregates.json`. KPIs that describe the full universe read from aggregates; lists and drill-downs read from rows.
4. **Hero records** (IDs in `DATA_SPEC.md`) always exist and sort to the top of relevant queues.
5. Market C has **no HCP-level sales**. Charts needing it show *"Not available in this market — using proxy: rep assessment + PMR"*. Never fabricate.
6. Fixed demo "today" = **2026-10-15**; all ages and dates are relative to it.
7. Formatting: thousands separators; % to 0–1 decimals; USD compact ($1.2M).

### 9.2 Files (`/data`)
| Group | Files |
|---|---|
| Reference | `markets.json`, `brands.json`, `personas.json`, `aggregates.json` |
| Standards | `dimensions.json`, `approval_rights.json`, `thresholds.json`, `stability_policy.json`, `vocabulary_map.json` |
| Master | `hcps.json`, `accounts.json`, `affiliations.json`, `consent.json` |
| Signals | `sales.json`, `crm_activity.json`, `digital_engagement.json`, `pmr_responses.json`, `formulary_status.json`, `kol_influence.json`, `rep_notes.json`, `feature_store.json` |
| Segmentation | `segment_assignments.json`, `segment_history.json`, `segment_library.json`, `studio_outputs.json` |
| Changes | `change_events.json`, `drift_flags.json`, `proposals.json` |
| Records | `audit_log.json`, `publish_log.json`, `downstream_impact.json`, `recalibration.json` |
| Intake | `vendor_file_mkt_c.json` (+ `vendor_file_mkt_c.xlsx` for upload) |
| AI | `ai_cache.json` |

Schemas, ID ranges and hero records are defined in `DATA_SPEC.md`.

---

## 10. AI layer (LLM-agnostic)

### 10.1 AI moments
| ID | Moment | Screen | Output |
|---|---|---|---|
| AI-1 | Vendor column & value mapping | S05 | source column → global field, value maps, confidence |
| AI-2 | Cluster naming & description | S06 | cluster id → name, description, global value |
| AI-3 | Explanation card | S10, D01 | headline, drivers[], confidence, data age, what would move it next |
| AI-4 | Ask Continuum (Q&A) | S15 | short answer + optional table + link to screen |
| AI-5 | Rep-note signal extraction | S09 Signal Feed, D01 | note → structured signal (dimension, direction, evidence quote, confidence) |

### 10.2 Architecture
- **`LLMProvider` interface:** `generate({ task, system, input, schema }): Promise<TaskOutput>`.
- **Adapters:** `MockProvider` (**default**, reads `ai_cache.json`) · `OpenAICompatibleProvider` (OpenAI, Azure OpenAI, Mistral, Groq, local Ollama/vLLM via `baseUrl`) · `AnthropicProvider` · `GeminiProvider` (optional). New providers = one new file.
- **Config** (`llmConfig.ts`): `provider`, `model`, `baseUrl`, `temperature` (0.2), `timeoutMs` (6000), `useProxy`. Settings panel (gear icon) lets the presenter switch provider at runtime.
- **Prompts** live in `/prompts/AI-1.md … AI-5.md`, provider-neutral, each with system instruction, input template and JSON schema.
- **Validation:** every response is parsed with its Zod schema. Invalid output, error or timeout → silently fall back to cached response. The demo never breaks.

### 10.3 Rules
- Screens and services **never import an LLM SDK** or call a model endpoint directly. Only `/src/llm/providers/*` may make network calls.
- **Demo mode (default):** `MockProvider`, no key, fully offline.
- **Live mode:** calls go via `/server/llm-proxy` which holds the key. For internal testing only, a key may be entered in Settings and held **in session memory only** (never stored, never in source, never in the build).
- Prompts instruct the model to: use only provided data/drivers; no promotional wording; no clinical claims; return valid JSON matching the schema.
- Every AI output shows an **AIBadge** ("AI-drafted · reviewed by owner before use") and the provider name in a tooltip.

---

## 11. State and interactivity

Session state lives in Zustand, seeded from `/data`, reset by **Demo reset** (top-bar menu).

- **Approve** → status `Approved` → audit entry (approver persona, timestamp, reason, before/after) → pipeline counters update (S09, S12) → record queued for publish (S11).
- **Reject** requires a reason: *Field knowledge contradicts signal · Temporary behaviour · Data quality issue · Freeze period · Other* → audit entry → feeds Recalibration (S13).
- **Hold** is applied automatically for conflicting signals, freeze windows or frequency limits; the triggering policy rule is shown.
- **Library:** a Studio run saves a *Draft* → Global Lead *Approves* → *Activates* (previous Active becomes *Retired*). Only the Active version can publish. **Bulk refresh** re-runs the Active version on new data with original parameters and produces proposals (never overwrites), shown as a Sankey.
- **Publish** writes approved changes to the mock CRM record (segment field updated, proposed-segment field cleared, segment age reset to 0) and updates downstream consumer counts.
- Cockpit, Segment Health and Pipeline KPIs recompute from state so numbers move during the demo.

---

## 12. Design system

**Look and feel:** calm, data-dense but uncluttered enterprise SaaS (Veeva / Salesforce Lightning / Microsoft Fluent quality). White canvas, navy chrome, generous spacing, one accent per screen. No gradients, illustrations or emojis.

### Colour tokens
| Token | Hex | Use |
|---|---|---|
| `navy-900` | `#0B2740` | Side nav, page titles, primary text |
| `navy-700` | `#13293D` | Headings, table headers |
| `slate-500` | `#5C7080` | Secondary text, labels |
| `slate-200` | `#CFE0EA` | Borders, dividers |
| `canvas` | `#F6F8FA` | App background |
| `teal-600` | `#0E7C86` | Primary actions, positive drift (▲), "Runs on its own" |
| `gold-500` | `#E9B44C` | Human gate, "Needs approval", guided-demo accents |
| `amber-700` | `#8A6D1F` | Approver labels, Hold state |
| `rust-600` | `#B0603C` | Warnings, medium confidence |
| `red-700` | `#B03A2E` | Negative drift (▼), errors, stale segments |
| `indigo-500` | `#5B6ABF` | Learning / model elements |

**AI-intervention badges** (match the architecture legend): Agent `#8A6D1F` · ML `#0E7C86` · GenAI `#5B6ABF` · Rules `#B0603C` · Human `#B03A2E`.
**Market colours:** A `#0E7C86`, B `#5B6ABF`, C `#B0603C`.
**Segment colours A→E:** `#0B2740`, `#1B4A63`, `#0E7C86`, `#9DB4C4`, `#CFE0EA`.

### Typography
Inter (fallback system-ui). Page title 22/600 · Section 16/600 · Body 14/400 · Table 13/400 · Label 12/500 uppercase tracking-wide.

### Layout
- Left nav 240px (collapsible to 64px) with group headers; top bar 56px; content max-width 1440px; 24px padding; 16px grid gap.
- Top bar: app name · Market selector · Brand selector · Persona switcher · Guided demo toggle · Settings · Demo reset.
- Page header: title, one-line question the screen answers, right-aligned primary action.
- Filter bar: Market · Brand · Indication · Customer type (HCP/Account) · Date range.
- Cards: white, 8px radius, 1px `slate-200` border, no heavy shadows.

### Standard components
- **KpiTile:** label, value, delta (▲/▼ coloured), optional sparkline.
- **StatusBadge:** Draft · Approved · Active · Retired · Proposed · Held · Rejected · Written back.
- **ConfidenceChip:** High (teal) · Medium (rust) · Low (red); "gap-filled" variant with dashed border.
- **SegmentAgeChip:** green <90d, amber 90–180d, red >180d.
- **InterventionBadge:** Agent · ML · GenAI · Rules · Human.
- **ApproverPill:** gold-outlined pill with the role that must approve.
- **AIBadge:** sparkle icon + "AI-drafted".
- Tables: sticky header, sortable, row hover, right-aligned numbers, row click opens D01.

### UX rules
- Every screen answers one question; show it under the title.
- Max 4 KPI tiles per row; max 2 primary charts above the fold.
- Every chart has a title, a one-line "so what" caption and axis labels.
- Designed empty, loading and "not available in this market" states — never blank.
- Governed actions use a confirmation modal showing before → after values and the approver.
- Toast after every action ("Proposal approved · queued for publish").

---

## 13. Glossary (use these exact terms in the UI)

| Term | Meaning |
|---|---|
| Segment age | Days since a segment value was last set or confirmed |
| Change event | A detected movement in a source signal (output of Signal watch) |
| Drift flag | Model/rule detection that evidence for a dimension has moved past the market threshold |
| Proposal | A drafted change to a governed field, awaiting owner decision |
| Hold | Proposal paused by stability policy (conflict, freeze, frequency limit) |
| Write-back | Approved value published to the CRM segment field |
| Global standard | Dimensions and allowed values defined by Global |
| Maturity level | Survey-led · Signal-enriched · Data-rich |
| Segment version | A saved segmentation run with method, parameters and status |
| Bulk refresh | Re-running the active version on new data with original parameters |
| Propagation | Account-level event triggering review of affiliated HCPs |
| Recalibration recommendation | Suggested threshold/model change from feedback; requires validation |

Pharma terms to use naturally: TRx, NRx, NBRx, sell-in, brick, PMR, KAM, IDN, formulary, tender, call plan, reach & frequency, omnichannel, consent, HCP, HCO.

---

## 14. Build rules for the AI coding agent

1. **Build only what the current prompt asks.** Do not modify screens or files outside the prompt's stated scope.
2. **Reuse shared components** from `/components`; create new ones only if none fits, and place them in `/components`.
3. **Respect this brief** for naming, colours, terms and principles. Do not invent personas, brands, markets, dimensions or screens.
4. **Do not import any LLM SDK.** All AI goes through `aiService.ts` → `LLMProvider`. Do not add Gemini-specific code outside `GeminiProvider.ts`.
5. **Do not generate or embed mock data in code.** Read from `/data/*.json` via `dataService.ts`.
6. No placeholder text ("Lorem ipsum", "TODO", "Sample") in the UI.
7. Type everything — no `any`. Shared types in `/src/types.ts`, matching `DATA_SPEC.md`.
8. Keep each screen file under ~400 lines; extract sub-components.
9. After each build step confirm: app compiles, existing routes render, hero path still works.
10. If ambiguous, choose the option that best supports the hero storyline and note it as `// ASSUMPTION:`.

---

## 15. Definition of done

- [ ] All 14 screens, D01 Customer 360 and the Ask Continuum panel render without errors.
- [ ] Persona switcher changes landing page, visible nav, label overrides and permitted actions (Section 5.1).
- [ ] Every reference-architecture component is visible (Section 6), and S09 Pipeline mirrors the architecture flow.
- [ ] Market and brand filters work on every dashboard.
- [ ] Hero path (Section 7) runs end-to-end in Guided demo mode with no dead ends.
- [ ] Approve / Reject / Hold / Publish update queue, audit log, pipeline, health KPIs and cockpit live.
- [ ] Market C never shows fabricated HCP-level sales.
- [ ] App runs fully offline with `MockProvider`; switching provider in Settings works when a proxy/key is supplied.
- [ ] No LLM vendor SDK in `package.json`.
- [ ] Demo reset restores the initial state.
- [ ] `npm run build` produces a static `dist/` that runs offline.
