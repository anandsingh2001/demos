# PROMPT_PACK.md — Continuum Screen Build Prompts

**Purpose:** Step-by-step build prompts for AI Studio / Antigravity. Run them in order, one per session turn. Each prompt builds on the one before it.

**Before you start**
- **AI Studio:** paste PROJECT_BRIEF.md into *System Instructions*. **Antigravity:** save it as `AGENTS.md` in the repo root.
- Put DATA_SPEC.md, GOVERNANCE_CONFIG.json and `/data/*.json` in the project before Prompt 1.
- Every prompt ends with the same **check**. If it fails, fix it before moving on.

**Standard check (paste at the end of every prompt if the tool doesn't keep context)**
```
CHECK: app compiles with no TypeScript errors; all existing routes render;
no data is hard-coded in screens (reads only via dataService.ts); no LLM SDK
imported; hero path still works. List the files you created or changed.
```

**Build sequence**

| Phase | Prompts | Output |
|---|---|---|
| A. Foundation | F1–F5 | Scaffold, design system, data + governance services, AI layer, app shell |
| B. Screens (hero order) | S01–S15, D01 | 14 screens, Customer 360, Ask Continuum |
| C. Wiring | W1–W3 | Live state loop, guided demo, persona QA |
| D. Hardening | H1–H2 | Edge states, final audit against Definition of Done |

> **Tip:** if a session drifts, start a fresh one with the brief in System Instructions and paste only the next prompt. Prompts are self-contained for that reason.

---

## Phase A — Foundation

### F1 · Scaffold
```
Create the Continuum project per PROJECT_BRIEF Section 8.
- React 18 + TypeScript + Vite, Tailwind, Recharts, d3-sankey, React Router,
  Zustand, Zod, SheetJS (xlsx), lucide-react. No LLM vendor SDKs.
- Create the full folder structure from Section 8 with empty, typed stubs.
- Tailwind theme: add every colour token from Section 12 (navy-900 … indigo-500),
  the AI-intervention, market and segment colours, Inter font, and type scale.
- appConfig.ts: appName "Continuum", tagline, demoToday "2026-10-15", feature flags.
- routes.tsx: a route for S01–S14, D01 (/customer/:type/:id), with placeholder
  pages showing only the screen title and its question from Section 5.
Do not build any screen content yet.
```

### F2 · Shared components
```
Build the shared components in /src/components per PROJECT_BRIEF Section 12.
ui/: KpiTile (label, value, delta ▲/▼ coloured, optional sparkline), DataTable
(sticky header, sortable, row hover, right-aligned numbers, onRowClick),
StatusBadge (Draft, Approved, Active, Retired, Proposed, Held, Rejected,
Written back), ConfidenceChip (High teal, Medium rust, Low red, gap-filled =
dashed border), SegmentAgeChip (<90d green, 90–180 amber, >180 red, label
"164 days"), InterventionBadge (Agent, ML, GenAI, Rules, Human — exact hex from
brief), ApproverPill (gold outline), AIBadge (sparkle + "AI-drafted", tooltip
"AI-drafted · reviewed by owner before use · Provider: {name}"), FilterBar,
Tabs, Drawer, Modal, ConfirmChangeModal (before → after + approver), EmptyState
(variants: empty, loading, notAvailableInMarket with the exact proxy message
from Section 9.1), Toast, StepIndicator.
charts/: Heatmap, Funnel, Sankey (d3-sankey), BoxPlot, Matrix4x4, Bar, Line,
Donut, PipelineFlow (horizontal stage cards with arrows). Every chart takes
title, soWhat caption and axis labels as required props.
Add a hidden /dev/components route that shows every component with sample props
so I can review the design system.
```

### F3 · Data and governance services
```
Implement the data layer per PROJECT_BRIEF Sections 9 and 11 and DATA_SPEC.md.
1. types.ts: interfaces for every /data file, matching DATA_SPEC.md exactly.
2. dataService.ts: the ONLY data access point. Loads /data/*.json, validates
   each with Zod on load (log, don't crash, on mismatch), and exposes typed
   query functions filtered by market, brand, indication, customer type, date.
   Universe KPIs read from aggregates.json; lists read from row files.
   Hero records (IDs in DATA_SPEC.md) sort to the top of queues.
   Market C sales queries return { available: false, proxy: "rep assessment + PMR" }.
3. governanceService.ts: canApprove(persona, market, dimension) from
   approval_rights.json; checkThresholds; checkStability (freeze window,
   change-frequency limit, min evidence, conflicting signals → Hold with the
   rule that triggered it).
4. useAppStore.ts (Zustand): persona, market, brand, filters, guidedStep,
   proposals, library versions, auditLog, publishLog, crmRecords; seeded from
   dataService; resetDemo() restores the seed.
5. Formatters: thousands separators, % to 0–1 dp, USD compact, age in days
   relative to demoToday.
Write unit tests for governanceService rules.
```

### F4 · LLM-agnostic AI layer
```
Implement the AI layer per PROJECT_BRIEF Section 10.
- LLMProvider interface: generate({ task, system, input, schema }).
- Providers: MockProvider (default; reads ai_cache.json keyed by task + input
  key), OpenAICompatibleProvider (baseUrl, model; covers OpenAI, Azure OpenAI,
  Mistral, Groq, Ollama, vLLM), AnthropicProvider, GeminiProvider (optional).
  Use fetch only. Only files in /src/llm/providers may make network calls.
- llmConfig.ts: provider, model, baseUrl, temperature 0.2, timeoutMs 6000, useProxy.
- schemas.ts: Zod schema for AI-1 … AI-5 per Section 10.1.
- aiService.ts: the ONLY way screens call AI. Parse with Zod; on invalid output,
  error or timeout, silently return the cached response. Return
  { data, provider, fromCache }.
- /prompts/AI-1.md … AI-5.md: system instruction, input template, JSON schema.
  Every prompt: use only provided data; no promotional wording; no clinical
  claims; return valid JSON only.
- /server/llm-proxy: optional ~50-line Node proxy reading the key from env.
- Settings panel (gear icon): choose provider, model, baseUrl, proxy on/off;
  optional session-only key field (memory only, never persisted). "Test
  connection" button.
```

### F5 · App shell, navigation and personas
```
Build AppShell, SideNav, TopBar and PageHeader per PROJECT_BRIEF Sections 5 and 12.
- SideNav 240px (collapsible to 64px), five groups with their questions as
  tooltips: Overview, Standards & Data, Segments, Changes, Controls.
- TopBar 56px: "Continuum" · Market selector (All markets, A, B, C from
  markets.json) · Brand selector (default Aurelix) · Persona switcher · Guided
  demo toggle · Settings · Demo reset.
- Persona switcher implements the Section 5.1 table exactly: landing page,
  visible nav, label override (S10 → "My Customers" for Field Rep), market scope
  (P2 default MKT_B, P5 locked to MKT_C), read-only for P6.
- Disabled actions show tooltip "Your role cannot approve this field in this market."
- PageHeader: title, one-line question, right-aligned primary action slot.
- Floating "Ask Continuum" button bottom-right on every screen (panel built later).
- About modal with the positioning line and the living segmentation definition
  from Section 1.
```

---

## Phase B — Screens

> Each prompt lists: **question · layout · data · actions · AI · acceptance**. Build screens in this order: it follows the hero storyline, so each part of the demo works as soon as it is built.

### S01 · Executive Cockpit
```
Build S01 Executive Cockpit. Question: "Is segmentation healthy and worth it?"
Layout:
- Row 1 KPI tiles (max 4): Segments current (<90 days) %, Median segment age,
  Proposals approved this quarter, Unapproved write-backs (must show 0, teal tick).
- Row 2 (2 charts): Segment age by market (stacked bar, green/amber/red bands);
  Living loop funnel (change events → drift flags → proposals → approved →
  written back) from store state.
- Row 3: Market maturity strip (A/B/C with maturity, process, refresh cadence,
  standardisation %); "Value to date" card linking to S02.
Data: aggregates.json + live store. Respect market and brand filters.
Read-only for every persona; Export (PNG/CSV) for P6.
Acceptance: numbers change after approvals/publish in later screens.
```

### S12 · Segment Health (Standardisation tab first for hero step 1)
```
Build S12 Segment Health. Question: "Are our segments current and trusted?"
Tabs:
1. Age – distribution histogram by market; table of oldest segments (row → D01).
2. Pipeline KPIs – events, flags, proposals, approval rate, hold rate, write-backs
   (live from store), trend lines.
3. Time to Update – median days from signal to write-back, living vs legacy
   cycle (bar), by market.
4. Field Trust – rep acceptance vs rejection rate, top rejection reasons, by market.
5. Standardisation – per market: % dimensions mapped to the global standard,
   process type (in-house / different tool / agency Excel), comparability score.
   Caption: "Three markets, three processes — segments can't be compared until
   mapped to the global standard."
Deep-linkable tabs (?tab=standardisation) for guided demo.
```

### S08 · Segment Insights
```
Build S08 Segment Insights. Question: "What do our segments tell us?"
Seven deep-linkable tabs:
1. Profiles – per segment A–E: count, share, avg potential, adoption stage mix,
   channel mix; segment colours from brief.
2. Opportunity – Matrix4x4 potential (1–4) × adoption; bubble = HCP count;
   highlight high-potential / low-adoption gap. Market C uses proxy, labelled.
3. Migration – Sankey of segment movement between last two versions.
4. Accounts – account archetype × tier table, access/formulary status, affiliated
   HCP counts; row → D01 Account. (Only tab visible to P3.)
5. Cross-market – same global segment side-by-side for A/B/C after mapping;
   before/after toggle "Local labels" vs "Global standard".
6. Cross-segmentation – HCP segment × channel affinity heatmap (Neurelle story).
7. Targeting alignment – current call plan vs recommended by segment; shows
   shift after publish (reads publishLog + downstream_impact).
Read-only for P6.
```

### S04 · Market Data
```
Build S04 Market Data. Question: "Is each market's data ready?"
Tabs:
1. Readiness – Heatmap: markets × data sources (survey/PMR, CRM activity & notes,
   sales/Rx, claims & access, digital, reference & affiliations); cell = available
   / partial / none, with maturity label per market. Caption explains why
   methods differ.
2. Data Quality – match rate to customer master, duplicate rate, affiliation
   coverage, completeness by field; per market.
3. Signal Freshness – each signal: last refresh, data age chip, expected cadence,
   late flag.
4. Feature Store – table: feature, source, refresh, data age, availability by
   market (A/B/C ticks).
P2 sees own market only.
```

### S05 · Data Intake
```
Build S05 Data Intake (hero step 3: onboard Market C). Question: "Can we bring
this market's file into the global standard?"
Four-step stepper:
1. Upload – drag-drop vendor_file_mkt_c.xlsx (SheetJS); preview first 20 rows.
   Provide "Use sample vendor file" button.
2. AI mapping (AI-1) – table: source column → global field, value maps (e.g.
   "Tier 1" → Segment A), ConfidenceChip, AIBadge; user can override any mapping.
3. Match & validate – match rate to customer master, unmatched list, invalid
   values, duplicates.
4. Conformance – % conformant to global standard; "Submit for approval" creates
   a Draft in the Segment Library.
P5 Local Agency: sees only this screen, can upload and view results, cannot
submit to library (button shows reason).
```

### S06 · Segmentation Studio
```
Build S06 Segmentation Studio (hero step 4). Question: "Build a segmentation
within my market's rights."
Four steps:
1. Filter – market, brand, indication, customer type, universe count.
2. Build – method picker limited by market: MKT_C rules template (potential grid
   with editable weights), MKT_A K-means (k, features from Feature Store), MKT_B
   rules/deciling + behavioural clusters. Results from studio_outputs.json.
3. Review – cluster profiles (BoxPlot, size bars); AI-2 names and describes each
   cluster with AIBadge; user can edit names.
4. Map to standard – map each cluster to a global segment value (A–E); warn on
   unmapped clusters.
"Save as version" → Draft in S07, audit entry, toast. Enforce rights via
governanceService (P2 own market only; P6 none).
```

### S07 · Segment Library
```
Build S07 Segment Library. Question: "Which segmentation versions do we have?"
- Table: version, market, brand, method, parameters, created by/date, status
  (StatusBadge), segments count, mapped-to-standard %.
- Lifecycle: Draft → Approve (P1 only) → Activate (previous Active → Retired).
  ConfirmChangeModal for each; audit entry; toast.
- Version detail drawer: parameters, segment distribution, diff vs active.
- "Bulk refresh" on the Active version: re-runs with original parameters on new
  data, produces PROPOSALS (never overwrites), and shows a Sankey of proposed
  moves with a "Send to Review Queue" button.
Only the Active version can publish.
```

### S09 · Change Monitor (signature screen)
```
Build S09 Change Monitor (hero step 6). Question: "What changed, and is it
material?"
Tab 1 Pipeline – implement PROJECT_BRIEF Section 6.1 exactly with PipelineFlow:
five stage cards (Signal watch → Drift detection → Explanation → Proposal &
routing → Record & learn → CRM) with description, InterventionBadges, live
count, output label; Stage 4 in gold with "on hold" sub-count. Clicking a stage
routes to its detail. Below: guardrails strip (AI proposes people decide · no
unapproved write-back · stability policy · consent respected · medical
firewall); feedback-loop arrow from stage 5 back to stage 2; "Who decides" band
(Runs on its own teal · Needs approval gold · Stays human-owned).
Tab 2 Signal Feed – change events list: time, customer, signal type, source,
magnitude, market. Hero events pinned on top: Zentrova formulary win at hero
account (MKT_B) with "Propagation: N affiliated HCPs" link; rep note for hero
HCP showing AI-5 extraction (dimension, direction, evidence quote, confidence)
with AIBadge.
Tab 3 Drift Flags – flags with dimension, threshold vs observed, ML confidence,
market threshold link to S03, status (→ Proposal / Hold with rule shown).
Counts filter by market and brand and update live.
```

### S10 · Review Queue
```
Build S10 Review Queue (hero step 6). Question: "What do I need to decide?"
- Queue table: customer, type (HCP/Account), dimension, current → proposed,
  ConfidenceChip, SegmentAgeChip, data age, ApproverPill, status. Hero HCP
  (Aurelix) and hero account propagation items on top.
- Filters: HCP / Account (P3 defaults to Accounts), market, brand, status
  (Proposed, Held, Approved, Rejected).
- Side panel on select: AI-3 explanation card (headline, drivers with ▲/▼,
  confidence, data age, "what would move it next") with AIBadge; source signals.
- Actions: Approve (ConfirmChangeModal), Reject (reason required from the five
  reasons in Section 11), Open Customer 360. Held items show the policy rule and
  cannot be approved.
- Bulk approve for selected High-confidence items (P2 only).
- Persona rules via governanceService: P4 Field Rep sees "My Customers", only
  own-territory HCPs, can act only on adoption stage and digital behaviour.
On approve/reject: audit entry, pipeline counters update, toast "Proposal
approved · queued for publish".
```

### D01 · Customer 360
```
Build D01 Customer 360 (HCP and Account variants), opened from any table row.
HCP: header (name, specialty, market, territory) + current segment with
SegmentAgeChip; all governed dimensions with current value, owner, last change;
segment history timeline; signals (sales — or Market C proxy message — CRM
activity, digital, PMR); rep notes with AI-5 extraction; consent & preference
panel (channel consent ticks); affiliations (→ account); open proposals with
AI-3 explanation and actions (same rules as S10).
Account: archetype, potential, access/formulary status (Zentrova win visible
for hero account), decision-making model, tier; affiliated HCPs with their
segments and propagation proposals.
Breadcrumb back to the originating screen.
```

### S11 · Publish to CRM
```
Build S11 Publish to CRM (hero step 7). Question: "What reaches the field, and
what changes downstream?"
- Ready-to-publish list (Approved only, Active version only) with select all.
- Mock CRM record view for the selected customer: segment field, proposed-segment
  field, approval workflow status, segment age — show before → after.
- "Publish" (P2/P3) → ConfirmChangeModal → updates crmRecords (segment set,
  proposed cleared, age reset to 0), publishLog entry, audit entry, toast.
- Downstream consumer tiles: Targeting, Call planning, NBA, Journeys,
  Segment-health dashboard — affected counts update after publish; audience
  counts exclude non-consented channels (show the exclusion number).
- Publish log table. Banner: "Unapproved write-backs: 0".
- Note on screen: "Continuum publishes to these systems; it replaces none of them."
```

### S13 · Audit & Learning
```
Build S13 Audit & Learning. Question: "Can we prove every change, and are we
learning safely?"
Tabs:
1. Change Log – every audit entry: timestamp, persona, action, customer,
   dimension, before → after, reason, source (AI-proposed / Studio / Intake).
   Filter + CSV export. Counter: unapproved write-backs = 0.
2. Version History – library lifecycle timeline per market/brand.
3. Recalibration – feedback loop inputs (approval rate, rejection reasons,
   reach, coverage of rising customers, overrides) → recalibration
   recommendation cards (e.g. raise MKT_B NBRx threshold). P1 can Validate or
   Decline; label "Recommendation — requires model-owner validation. No
   auto-retraining." Live rejections from S10 feed these numbers.
```

### S14 · Responsible AI
```
Build S14 Responsible AI. Question: "Is it fair and under control?"
- Bias check: segment and proposal rates by urban/rural, specialty, gender
  (Brevanta GP + specialist story); parity ratio with a tolerance band; flag
  out-of-band groups.
- AI usage register: AI-1 … AI-5 — purpose, screen, human check, provider in
  use, fallback rate (from aiService), last test date.
- Guardrails status: no promotional language check, no clinical claims, medical
  firewall, consent respected, no auto-retrain — each with a pass tick.
- Model cards for drift classifier: features, thresholds per market, confidence
  calibration chart.
Read-only for all; export for P6.
```

### S02 · Value Calculator
```
Build S02 Value Calculator. Question: "What is living segmentation worth?"
- Inputs (sliders, defaults from aggregates.json): HCP universe, % rising
  customers missed by annual refresh, value per reached HCP, call cost, analyst
  days per refresh per market, number of markets.
- Outputs: value from earlier capture of rising customers, wasted calls avoided,
  analyst effort saved, time-to-update reduction — KpiTiles + waterfall bar.
- Toggle "Use demo actuals" pulls live approvals/write-backs from the store.
- Assumptions panel listing every formula in plain language. No promotional
  wording; label as "illustrative estimate".
```

### S03 · Global Standards
```
Build S03 Global Standards (P1 landing page). Question: "What are the global
rules?"
Tabs:
1. Dimensions – dimension catalogue: dimension, HCP/Account, allowed values,
   owner, version; filter by brand and market.
2. Approval Rights – matrix: dimension × market → approving persona
   (ApproverPill); editable by P1 only with audit entry.
3. Thresholds – per market and dimension: signal, threshold, min evidence,
   cadence; editable by P1/P2 (own market) with ConfirmChangeModal.
4. Stability Policy – freeze windows (calendar), change-frequency limits,
   minimum evidence, conflict rule → Hold. Show "Proposals held by this rule: N".
Edits flow into governanceService live.
```

### S15 · Ask Continuum
```
Build S15 Ask Continuum as a floating right-side panel (AI-4), available on
every screen.
- Suggested questions based on current screen (e.g. on S10: "Why is the hero HCP
  proposed to move to Segment B?"; on S01: "Which market has the oldest segments?").
- Answer: short text + optional small table + "Open in {screen}" link that
  routes with filters set. AIBadge with provider tooltip.
- Grounding: pass only the relevant data slice from dataService; answers stay
  within the data; no promotional or clinical language.
- MockProvider: cached answers for at least 10 suggested questions; any other
  question returns a polite "I can answer questions about segments, changes,
  data readiness and governance in this demo" plus suggestions.
```

---

## Phase C — Wiring

### W1 · Live state loop
```
Wire the end-to-end state loop per PROJECT_BRIEF Section 11. Verify and fix:
Approve in S10/D01 → audit entry → S09 counts → S12 KPIs → S01 cockpit → S11
ready list. Reject → reason → S13 Recalibration inputs. Hold rules from S03
take effect immediately. Publish → CRM record, downstream tiles, S08 Targeting
alignment, publish log. Library activation changes what can publish.
Demo reset restores every number. Write a short test script that runs the hero
approve → publish sequence and asserts the counters.
```

### W2 · Guided demo
```
Implement the Guided demo toggle per PROJECT_BRIEF Section 7.
- StepIndicator (1–8) in the top bar with step title, Back / Next.
- Each step sets persona, market, brand, filters and route (with tab query):
  1 P6 · S01 → S12?tab=standardisation → S08?tab=cross-market
  2 P1 · S04?tab=readiness → data-quality
  3 P5 · MKT_C · S05
  4 P2/P1 · MKT_C then MKT_A · S06 → S07 → S08?tab=profiles
  5 P2 · S08?tab=opportunity → accounts → cross-segmentation
  6 P2 · MKT_B · Zentrova then Aurelix · S09 pipeline → signal-feed → S10 → D01 hero HCP
  7 P2 · S11 → S08?tab=targeting
  8 P6/P1 · S12 → S13 → S14 → S02 → S01
- Sub-steps inside a step advance with Next; a gold outline highlights the
  element to click (e.g. hero row). Presenter notes (1–2 lines) in a collapsible
  tooltip.
- Exiting the guided demo keeps current state.
```

### W3 · Persona QA
```
For each persona P1–P6, walk every visible screen and verify against
PROJECT_BRIEF Section 5.1 and approval_rights.json: landing page, visible nav,
label overrides, market scope, enabled vs disabled actions with the correct
tooltip, read-only for P6. Output a PASS/FAIL matrix (persona × screen) and fix
every FAIL.
```

---

## Phase D — Hardening

### H1 · States and polish
```
On every screen add designed loading, empty and "not available in this market"
states (never blank). Check: max 4 KPI tiles per row, max 2 charts above the
fold, every chart has title + so-what caption + axis labels, segment age shown
wherever a segment appears, AIBadge on every AI output, glossary terms used
exactly (Section 13), no placeholder text, consistent formatting. Check 1280px
and 1440px widths. Keep each screen file under ~400 lines.
```

### H2 · Definition of done audit
```
Audit the app against PROJECT_BRIEF Section 15 item by item and Section 6
(every architecture component visible). Also confirm: Market C never shows
HCP-level sales; package.json has no LLM vendor SDK; app runs offline with
MockProvider; switching provider in Settings works when a proxy/key is
supplied; npm run build produces a static dist/ that runs offline.
Output a checklist with PASS/FAIL and evidence (file/screen), then fix FAILs.
```

---

## Fix-up prompts (use as needed)

| Situation | Prompt |
|---|---|
| Agent changed other screens | `Revert changes outside {screen}. Build only what this prompt asks (brief Section 14).` |
| Hard-coded data appeared | `Move all data in {file} to dataService.ts queries reading /data. Screens must not contain data.` |
| Design drift | `Restyle {screen} to the Section 12 tokens and shared components only. No new colours, gradients or shadows.` |
| AI call broke the screen | `Route this through aiService.ts with Zod validation and silent cache fallback. The demo must never break.` |
| Hero record missing | `Ensure hero IDs from DATA_SPEC.md appear at the top of {screen} for the guided-demo filters.` |
| Screen too long | `Split {screen} into sub-components under /screens/{screen}/ so each file is under 400 lines. No behaviour change.` |
