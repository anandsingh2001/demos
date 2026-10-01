DATA_PROMPT_PACK.md — Synthetic Data for the Living Segmentation Demo
Purpose: Run these prompts in sequence in AI Studio / Antigravity (or any LLM) to generate a coherent, linked synthetic dataset for the HCP / account segmentation demo.
How to use
Paste Prompt 0 first. It sets shared rules for the whole session.
Run Prompts 1–9 in order. Each one builds on tables created before it.
Run Prompt 10 last to validate the full set.
Save outputs to `/data/` using the file names given.
> **Design choice:** Each prompt asks the model to write a **seeded generator script** (TypeScript, runs in the browser or Node), not to type out rows directly. This keeps data reproducible, internally consistent, and easy to resize. It also keeps the app LLM-agnostic, because the data does not depend on a live model.
---
Prompt 0 — Master context (paste once, at the start)
```
You are a synthetic data engineer for a pharma commercial analytics demo called
"Living Segmentation" (HCP and account segmentation that updates continuously
instead of once a year).

Global rules for every prompt in this session:
- All data is fictional. No real HCP names, NPIs, companies, products or brands.
- Write a deterministic TypeScript generator (seeded PRNG, seed = 42) that writes
  CSV files to /data/. Also export the same rows as typed JSON for the app.
- Use stable IDs and keep foreign keys consistent across all tables:
  MKT-xx, TER-xxxx, HCP-xxxxxx, ACC-xxxxx, PRD-xx, SEG-xx, RUN-xxxx.
- Time window: 24 months, Oct 2024 – Sep 2026, monthly grain unless stated.
- Therapy area: Cardiometabolic. Brands: "Cardivex" (established),
  "Glyronal" (growth), "Nephrova" (launch, Apr 2026).
- 3 markets with different data maturity (key demo story):
    MKT-01 "Market A" – data-rich: full Rx, digital, CRM. ~1,200 HCPs
    MKT-02 "Market B" – mid: CRM + panel sales, partial digital. ~700 HCPs
    MKT-03 "Market C" – data-poor: CRM + rep input only, no HCP-level Rx. ~400 HCPs
- Distributions must be realistic: long-tail (Pareto) prescribing, seasonality,
  missing values where a market lacks a source, and believable noise.
- After the code, print: row counts per file, column dictionary, and 3 sample rows.
```
---
Prompt 1 — Reference data
```
Generate reference tables:
1. markets.csv: market_id, market_name, region, data_maturity (High/Medium/Low),
   available_sources (pipe-separated), currency.
2. territories.csv: territory_id, market_id, territory_name, region_name,
   rep_count. 12 / 8 / 5 territories for Markets A / B / C.
3. products.csv: product_id, brand_name, lifecycle_stage
   (Established/Growth/Launch), launch_date, indication.
4. specialties.csv: specialty_code, specialty_name, is_core_target (bool).
   Include Cardiology, Endocrinology, Nephrology, Internal Medicine, GP/FM,
   Diabetology, Other.
```
Prompt 2 — HCP master
```
Generate hcps.csv (~2,300 rows across 3 markets):
hcp_id, market_id, territory_id, first_name, last_name (fictional, locally
plausible), specialty_code, sub_specialty, years_in_practice, gender,
practice_setting (Hospital/Clinic/Private/IDN-affiliated), city, urban_rural,
kol_tier (None/Local/National; ~5% non-None), digital_consent (bool),
preferred_channel (F2F/Remote/Email/None), is_active, created_date.
Rules: ~40% GP/FM, ~20% Cardiology, ~15% Endocrinology, rest spread.
Market C: 20% of sub_specialty and preferred_channel are null.
```
Prompt 3 — Accounts and affiliations
```
Generate:
1. accounts.csv (~450 rows): account_id, market_id, territory_id,
   account_name (fictional), account_type (Hospital/IDN/Group Practice/
   Clinic/Pharmacy), bed_count (hospitals only), formulary_status per brand
   (Preferred/Listed/Not Listed), parent_account_id (IDN hierarchy, 2 levels),
   decision_making_unit_size.
2. hcp_account_affiliations.csv: hcp_id, account_id, role
   (Prescriber/Decision Maker/Influencer/Admin), is_primary, start_date.
Rules: every HCP has 1 primary affiliation; ~30% have 2–3 affiliations.
Each IDN has at least one Decision Maker.
```
Prompt 4 — Prescribing and sales (monthly)
```
Generate:
1. hcp_rx_monthly.csv – ONLY for Market A: month, hcp_id, product_id, trx, nrx,
   market_trx (total class), share = trx / market_trx.
2. account_sales_monthly.csv – Markets A and B: month, account_id, product_id,
   units, net_sales.
3. Market C: no Rx file. Create rep_potential_estimates.csv: hcp_id,
   estimated_potential (Low/Med/High), estimate_date, rep_confidence (1–5).
Rules: Pareto (top 20% of HCPs drive ~70% of volume); Q4 dip, Q1 rebound;
Nephrova starts at 0 in Apr 2026 and ramps on an S-curve, with uptake led by
Nephrology and KOL HCPs. Plant 3 behaviour shifts for the demo:
  (a) ~60 Market A Endocrinologists switch from Cardivex to Glyronal from Jan 2026;
  (b) one Market B IDN loses Cardivex formulary status in Mar 2026;
  (c) ~40 early Nephrova adopters stand out by Jun 2026.
Log these in planted_events.csv (event_id, description, affected_ids, start_month).
```
Prompt 5 — Field and digital engagement
```
Generate:
1. crm_interactions.csv: interaction_id, date, hcp_id, account_id, rep_id,
   channel (F2F/Remote/Phone), product_discussed, duration_min, call_outcome,
   sample_dropped (bool).
2. digital_engagement.csv (Market A full, Market B ~50% coverage, Market C none):
   event_id, date, hcp_id, channel (Email/Web/Webinar/Portal), content_id,
   action (Sent/Opened/Clicked/Attended/Downloaded).
3. events_attendance.csv: event_id, date, hcp_id, event_type
   (Congress/Symposium/Advisory Board/Speaker Program), product.
Rules: call frequency matches the old static segment (high-potential HCPs get
more calls), so engagement looks misallocated once living segments diverge.
Email open rate ~22%, click rate ~3%. Rep IDs map to territories.
```
Prompt 6 — Legacy static segmentation (baseline)
```
Generate legacy_segments.csv: hcp_id, market_id, legacy_segment
(A/B/C/D), legacy_decile, assigned_date (one annual run, Jan 2025),
method ("Annual decile on prior-year volume").
This is the "as-is" baseline. It must NOT reflect the planted shifts from
Prompt 4, so the demo can show where the static view becomes stale.
```
Prompt 7 — Living segmentation outputs
```
Generate the outputs of the living segmentation engine:
1. segment_definitions.csv: segment_id, segment_name, description, priority,
   recommended_cadence. Use 6 segments, e.g. "High-Value Loyalists",
   "Growth Switchers", "Launch Early Adopters", "Access-Constrained",
   "Digitally Engaged Low-Touch", "Low Potential / Maintain".
2. hcp_segment_snapshots.csv – monthly, Jan 2025 to Sep 2026: snapshot_month,
   hcp_id, segment_id, potential_score (0–100), engagement_score (0–100),
   access_score (0–100), confidence (0–1), top_3_drivers (pipe-separated,
   readable text, e.g. "NRx Glyronal +35% QoQ").
3. segment_transitions.csv: hcp_id, from_segment, to_segment, transition_month,
   trigger_type (Behaviour/Access/Engagement/Data refresh), explanation.
4. account_segment_snapshots.csv – same idea at account level.
Rules:
- The planted shifts from Prompt 4 must appear as transitions in the right months.
- Monthly churn between segments: 3–6%; most HCPs stay stable.
- Confidence is lower in Market C (0.4–0.65) than Market A (0.75–0.95).
  Market C scores rely on CRM and rep estimates (proxy features).
- ~15% of HCPs: legacy segment ≠ current living segment (the "stale gap").
```
Prompt 8 — Model runs and process KPIs
```
Generate:
1. model_runs.csv: run_id, run_date (weekly), market_id, model_version,
   features_used, hcps_scored, segments_changed, drift_flag (bool),
   psi_score, silhouette_score, runtime_min, status (Success/Warning/Failed).
   Include 2 warnings and 1 failed run, plus a model_version bump in Apr 2026.
2. process_kpis_monthly.csv: month, market_id, segmentation_cycle_time_days
   (legacy ~90 → living ~7), pct_hcps_with_fresh_segment, data_completeness_pct,
   field_adoption_pct (reps acting on segment updates), call_alignment_pct
   (calls to priority segments), override_rate_pct.
Rules: values improve after the "go-live" month (Jan 2026); Market C improves
least and stays below Markets A and B.
```
Prompt 9 — Data quality and feedback loop
```
Generate:
1. data_quality_issues.csv: issue_id, detected_date, market_id, source,
   table_name, issue_type (Missing/Duplicate/Outlier/Late feed/Mapping),
   records_affected, severity, status (Open/Resolved).
   Include 5–10 duplicate HCPs in Market B with matching name and city.
2. rep_feedback.csv: feedback_id, date, rep_id, hcp_id, suggested_segment,
   system_segment, reason (free text, 1 sentence), accepted (bool).
   ~200 rows, ~35% accepted; more feedback in Market C (rep knowledge fills
   data gaps).
```
Prompt 10 — Validation (run last)
```
Write validate.ts that checks all /data files and prints a PASS/FAIL report:
- Primary keys unique; all foreign keys resolve.
- Every HCP has a primary affiliation, a legacy segment and a snapshot every
  month from Jan 2025.
- Market A only in hcp_rx_monthly; no digital rows for Market C.
- Planted events appear in segment_transitions within ±1 month.
- Pareto check: top 20% HCPs = 65–75% of Market A TRx.
- Confidence by market: A > B > C.
- Nephrova volume = 0 before Apr 2026.
Then print a short summary of the demo stories the data supports.
```
---
Output files at a glance
#	File	Feeds dashboard
1	markets, territories, products, specialties	Filters, all views
2	hcps	Input data exploration
3	accounts, hcp_account_affiliations	Account view, IDN hierarchy
4	hcp_rx_monthly, account_sales_monthly, rep_potential_estimates, planted_events	Input exploration, drivers
5	crm_interactions, digital_engagement, events_attendance	Engagement / call alignment
6	legacy_segments	Static vs living comparison
7	segment_definitions, hcp/account_segment_snapshots, segment_transitions	Segmentation output analysis
8	model_runs, process_kpis_monthly	Process KPIs, model health
9	data_quality_issues, rep_feedback	Data quality, human-in-the-loop
Demo stories built into the data
Static segments go stale: Endocrinologists moving to Glyronal are still marked "Maintain" in the legacy view.
Access shock: an IDN loses formulary status, and its HCPs move to "Access-Constrained" within a month.
Launch early adopters: Nephrova adopters show up by Jun 2026.
Works in low-data markets: Market C runs on proxies and rep input, and shows lower confidence clearly.
Faster cycle: segmentation refresh drops from ~90 days to ~7 days.
