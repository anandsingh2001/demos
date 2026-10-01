DATA_SPEC.md — Continuum mock data specification
> **Purpose.** Defines every mock data file in `/data`: schema, keys, allowed values, volumes, distributions, hero records and metric formulas. The app reads these files through `dataService.ts` only.
> **Companion to:** `PROJECT_BRIEF.md` (v2). If they conflict on data matters, this file wins.
> **Database-ready.** Each file = one table. Field names are `snake_case`, keys are explicit, records are flat. Moving to a database later changes only `dataService.ts`.
---
1. Conventions
Item	Rule
File format	UTF-8 JSON array of flat objects (except `aggregates.json`, `ai_cache.json`, `stability_policy.json` — objects)
Demo "today"	2026-10-15. No date after today except freeze windows and scheduled refreshes
Dates	ISO `YYYY-MM-DD`; timestamps ISO `YYYY-MM-DDTHH:mm:ssZ`
Money	USD, number (no symbols); app formats
Percent	Decimal 0–1 (e.g. `0.124`); app formats
Nulls	`null` = not available in this market. Never `0` to mean "unknown"
Booleans	`true` / `false`
Enums	Exactly as listed in this spec (case-sensitive)
Names	Fictitious only. No real people, institutions or products
1.1 Codes
Entity	Codes
Market	`MKT_A`, `MKT_B`, `MKT_C`
Brand	`AUR` Aurelix · `ZEN` Zentrova · `CRD` Cardivance · `NEU` Neurelle · `BRV` Brevanta
Indication	`AUR_PSO`, `AUR_PSA`, `ZEN_NSCLC`, `ZEN_HER2`, `CRD_HF`, `CRD_CKD`, `NEU_MIG`, `NEU_MS`, `BRV_SA`, `BRV_COPD`
Persona	`P1`–`P6`
1.2 ID formats and ranges
Entity	Format	MKT_A	MKT_B	MKT_C
HCP	`HCP-{m}-{0000}`	`HCP-A-0001`–`0600`	`HCP-B-0001`–`0400`	`HCP-C-0001`–`0250`
Account	`ACC-{m}-{000}`	`ACC-A-001`–`120`	`ACC-B-001`–`080`	`ACC-C-001`–`040`
Territory	`TER-{m}-{00}`	`TER-A-01`–`12`	`TER-B-01`–`08`	`TER-C-01`–`05`
Brick (MKT_B only)	`BRK-B-{000}`	—	`BRK-B-001`–`060`	—
User	`USR-{000}`	global		
Other records	`{PREFIX}-{000000}`	e.g. `EVT-000001`, `DRF-000001`, `PRP-000001`, `AUD-000001`, `PUB-000001`, `NOTE-000001`, `VER-0001`		
Hero records always use the lowest IDs in their range (e.g. `HCP-B-0001`) so they sort first.
---
2. Volumes
Row files hold a representative sample; `aggregates.json` holds full-universe figures.
	MKT_A	MKT_B	MKT_C	Total rows
Full universe HCPs / Accounts (aggregates only)	8,000 / 900	4,000 / 400	1,500 / 150	—
`hcps.json` rows	600	400	250	1,250
`accounts.json` rows	120	80	40	240
`affiliations.json`	~900	~600	~330	~1,830
`crm_activity.json` (last 12 months)	~7,000	~5,500	~2,500	~15,000
`digital_engagement.json` (monthly)	600×12	400×12 (consented only)	~60×12	~12,700
`sales.json`	HCP×brand×month	brick×brand×month	account sell-in×quarter	see §5
`change_events.json`	~380	~260	~110	~750
`drift_flags.json`	~110	~75	~35	~220
`proposals.json`	~70	~50	~25	~145
Full-universe pipeline counts shown on screens (e.g. 1,240 events) come from `aggregates.json`, not from counting rows.
---
3. Hero records
These records drive the demo storyline and must exist exactly as specified. Values not listed follow normal generation rules.
3.1 People (users)
user_id	Name	Persona	Market	Notes
USR-001	Elena Marsh	P1 Global Segmentation Lead	ALL	
USR-002	Jonas Weber	P2 Market Back Office	MKT_B	default P2
USR-003	Sofia Lindqvist	P2 Market Back Office	MKT_A	
USR-004	Arif Santoso	P2 Market Back Office	MKT_C	
USR-005	Priya Raman	P3 KAM / Account Lead	MKT_B	owns ACC-B-001
USR-006	Tomás Ferreira	P4 Field Rep	MKT_B	territory TER-B-03; covers HCP-B-0001
USR-007	Dewi Kartika	P5 Local Agency	MKT_C	agency: "Northbay Field Research"
USR-008	Catherine Doyle	P6 Executive (CCO)	ALL	
USR-009	Marcus Okafor	P6 Compliance	ALL	
3.2 Hero account — formulary win (propagation)
Field	Value
hco_id	ACC-B-001
name	St. Aldric University Hospital
type	IDN / hospital group
market	MKT_B, region "North", urban
Zentrova formulary event	Status `Under review` → `Listed` on 2026-10-08 (tender outcome)
Account proposals	Zentrova access status `Under review → Listed`; account tier `Tier 2 → Tier 1` (approver: KAM)
Affiliated HCPs	HCP-B-0001 (dermatology, Aurelix hero) and HCP-B-0002 … HCP-B-0006 (oncology)
Propagation	Event creates 5 Zentrova proposals for HCP-B-0002…0006: potential up one level (e.g. 2→3) and segment up one level; confidence High for 0002–0004, Medium for 0005, Hold for 0006 (conflicting signal: declining call acceptance)
3.3 Hero HCP — living loop (Aurelix)
Field	Value
hcp_id	HCP-B-0001
name	Dr. Hanna Vogel
specialty	Dermatology · Specialist
primary account	ACC-B-001 · territory TER-B-03 · brick BRK-B-014
brand / indication	Aurelix · Plaque psoriasis
segment set on	2026-05-04 → segment age 164 days
Current (CRM) → Proposed	Segment B → A · Potential 3 → 3 (no change) · Behavioural Traditionalist → Engaged · Attitudinal Late majority → Early adopter · Digital affinity Evolving → Savvy · Adoption stage Consideration → Expansion
Approvers	Segment, Potential, Behavioural: Back Office · Attitudinal: Survey + Back Office · Digital, Adoption: Rep
Drivers	Brick-level Aurelix sales +12% QoQ (BRK-B-014) · portal visits 3 → 6 per month · latest PMR wave (Sep 2026) response shifted to "early adopter" · rep note NOTE-000001
Confidence	High (4 independent signals)
Change events	EVT-000001 (sales), EVT-000002 (digital), EVT-000003 (PMR), EVT-000004 (rep note, GenAI-extracted)
Drift flags	DRF-000001 … DRF-000005
Proposals	PRP-000001 … PRP-000005 (status `Proposed`)
Hero rep note (NOTE-000001, 2026-10-10, Tomás Ferreira):
> "Dr. Vogel has started 4 new moderate-to-severe psoriasis patients on Aurelix since August and asked for the patient support programme materials. Prefers to receive updates through the portal rather than in-person visits."
AI-5 extracts: adoption stage → Expansion (High), digital affinity → Savvy (Medium), channel preference → Portal (Medium).
3.4 Supporting hero cases
ID	Case	Purpose in demo
HCP-B-0007	Aurelix proposal Held: sales up but rep note says temporary sample-driven trial	Conflicting-signals rule
HCP-A-0001	Aurelix proposal Held: freeze window (MKT_A incentive period 2026-10-01 → 2026-10-31)	Freeze rule
HCP-A-0002	Previously Rejected proposal (reason: Temporary behaviour)	Rejection analysis
HCP-A-0003	K-means hero: moves `cluster_2` → mapped "Engaged / Segment A"	Studio mapping
HCP-C-0001	Market C: rep-observation-driven proposal (Adoption Consideration → Trial), confidence Medium, gap-filled potential	Survey-led loop
HCP-B-0010	Cardivance: sales signal below threshold → change event, no drift flag	"Not every signal changes the segment"
HCP-B-0020	Neurelle: high digital, low call reach	Cross-segmentation priority cell
HCP-A-0100…0140	Brevanta rural GPs under-represented in Segment A	Bias alert
3.5 Hero vendor file (Market C)
`vendor_file_mkt_c.json` — 260 rows from "Northbay Field Research", Aurelix + Zentrova, deliberately messy:
Issue	Count
Local column names (`Dr_Name`, `Spec`, `Clinic`, `Pot_Class`, `Segmen`, `Adopt`, `Last_Visit`)	7 columns
Non-standard values: Potential `High/Med/Low`; Segment `Gold/Silver/Bronze`; Adoption `Aware/Trying/Using/Loyal`	—
Missing customer ID	18 rows
Typos in name/specialty	12 rows
Duplicates	9 rows
HCPs not in customer master	11 rows
Expected outcome after mapping	match rate 92.3% (240/260); conformance 88%; data-quality score 74 → 91 after fixes
---
4. Reference and standards files
4.1 `markets.json`
Field	Type	Notes
market_code	string	MKT_A/B/C
display_name	string	"Market A · Data-rich"
maturity_level	enum	`Data-rich` · `Signal-enriched` · `Survey-led`
profile	string	one line
current_process	string	
current_tool	string	A: "In-house Python model" · B: "Third-party segmentation tool" · C: "Agency Excel file"
studio_method	enum	`KMEANS_RULES` · `RULES_DECILE_CLUSTERS` · `RULES_TEMPLATE`
refresh_cadence	enum	`Quarterly` · `Annual`
refresh_cycles_per_year	int	4 / 1 / 1
manual_hours_per_refresh	int	120 / 260 / 340
vendor_dependent	bool	false / false / true
on_global_standard	bool	false / false / false (becomes true in demo after mapping)
brands	string[]	A,B: all 5 · C: AUR, ZEN, CRD
hcp_universe	int	8000 / 4000 / 1500
account_universe	int	900 / 400 / 150
currency_note	string	"Values shown in USD"
4.2 `brands.json`
`brand_code, brand_name, therapy_area, lifecycle (Launch|Growth|Mature), launch_date, indications [{indication_code, name}], markets [], key_specialties []`
Brand	Key specialties
AUR	Dermatology, Rheumatology
ZEN	Medical oncology, Thoracic oncology, Breast surgery
CRD	Cardiology, Nephrology, General practice
NEU	Neurology, Headache specialist
BRV	Pulmonology, Allergy, General practice
4.3 `personas.json`
`persona_id, persona_name, user_id, market_scope, landing_screen, visible_screens [], label_overrides {}, can_approve_dimensions []` — values per `PROJECT_BRIEF.md` §5.1.
4.4 `dimensions.json` (dimension catalogue)
Field	Type	Notes
dimension_code	string	see below
customer_type	enum	`HCP` · `HCO`
name	string	
captured_per	enum	`brand_indication` · `brand` · `customer` · `brand_market`
allowed_values	string[]	global vocabulary
layer	enum	`Strategic` · `Priority` (per POV decision layers)
refreshable_between_cycles	bool	
global_fixed	bool	true = markets cannot change values
market_configurable	string[]	e.g. `["thresholds","cadence","micro_segment_scheme"]`
version	string	`v3.2`
effective_from	date	

dimension_code	Type	Allowed values	Layer	Refreshable
SEGMENT	HCP	A, B, C, D, E	Priority	true
POTENTIAL	HCP	1, 2, 3, 4	Priority	true
ADOPTION	HCP	Unaware, Aware, Consideration, Trial, Adoption, Expansion	Priority	true
BEHAVIOURAL	HCP	Traditionalist, Pragmatist, Engaged, Advocate	Strategic	false (cycle only; flags for review)
ATTITUDINAL	HCP	Early adopter, Early majority, Late majority, Sceptic	Strategic	true (between survey waves)
DIGITAL	HCP	Traditional, Evolving, Savvy	Priority	true
CHANNEL_PREF	HCP	Face-to-face, Remote, Email, Portal, Events, Mixed	Priority	true
MICRO_SEGMENT	HCP	market scheme (e.g. MS-01…MS-08) or `None`	Priority	true
ACC_ARCHETYPE	HCO	Academic centre, Community hospital, Specialty clinic, Integrated network, Private practice group, Pharmacy chain	Strategic	false
ACC_POTENTIAL	HCO	1, 2, 3, 4	Priority	true
ACCESS_STATUS	HCO	Listed, Restricted, Under review, Not listed	Priority	true
DECISION_MODEL	HCO	Clinician-led, Committee-led, Protocol-driven, Procurement-led	Strategic	false
TREATMENT_CAP	HCO	Full (infusion + specialist), Partial, Referral only	Strategic	false
ACC_TIER	HCO	Tier 1, Tier 2, Tier 3	Priority	true
4.5 `approval_rights.json` (maintenance-rights matrix)
`market_code, dimension_code, maintained_by_today, approver_role (P2|P3|P4|P1), ai_role (Propose|Detect drift|Infer and suggest|Refresh between waves|Flag for review), never (string), co_approver_role (nullable)`
Defaults (all markets, unless overridden):
Dimension	Approver	AI role	Never
SEGMENT, POTENTIAL, MICRO_SEGMENT	P2 Back Office	Propose	overwrite
BEHAVIOURAL	P2 Back Office	Flag for review	change between cycles
ATTITUDINAL	P2 Back Office (+ survey evidence required)	Refresh between waves	replace the survey
ADOPTION, DIGITAL, CHANNEL_PREF	P4 Rep	Detect drift / Infer and suggest	change silently
ACC_TIER, ACC_POTENTIAL, ACCESS_STATUS	P3 KAM	Propose	overwrite
ACC_ARCHETYPE, DECISION_MODEL, TREATMENT_CAP	P3 KAM (+ P1 co-approve)	Flag for review	change between cycles
Overrides: MKT_C POTENTIAL approver = P4 Rep (+ P2 co-approve) — reps maintain potential in survey-led markets.
4.6 `thresholds.json`
`market_code, dimension_code, signal_type, threshold_value, unit, min_independent_signals, min_confidence (High|Medium|Low), lookback_days`
Market	Example	Value
MKT_A	SEGMENT · HCP TRx change	±15% QoQ, min 2 signals, min Medium, 90 days
MKT_B	SEGMENT · brick sales change	±10% QoQ, min 2 signals, min Medium, 90 days
MKT_C	ADOPTION · rep observation	1 structured rep assessment + 1 CRM signal, min Medium, 120 days
4.7 `stability_policy.json` (object, keyed by market)
```json
{
  "MKT_A": {
    "max_changes_per_customer_per_180d": 2,
    "freeze_windows": [{"name": "Q4 incentive period", "start": "2026-10-01", "end": "2026-10-31"}],
    "conflict_rule": "hold_if_signals_disagree",
    "min_days_between_changes": 60,
    "stable_dimensions": ["BEHAVIOURAL","ACC_ARCHETYPE","DECISION_MODEL","TREATMENT_CAP"]
  },
  "MKT_B": { "...": "freeze window 2026-12-01 → 2026-12-31 (planning)" },
  "MKT_C": { "...": "freeze window 2027-01-01 → 2027-01-20 (territory alignment)" }
}
```
4.8 `vocabulary_map.json`
`market_code, source (Studio|Vendor|Legacy tool), source_dimension, source_value, global_dimension, global_value, mapped_by (AI|User), confidence, status (Accepted|Pending|Rejected)`
Must include: MKT_A `cluster_0…cluster_4` → SEGMENT/BEHAVIOURAL values; MKT_B legacy tool tiers `T1/T2/T3/T4` → SEGMENT A–D; MKT_C vendor `Gold/Silver/Bronze` → A/B/C and `High/Med/Low` → 4/3/2 (and `Low` → 1 when volume proxy is lowest decile).
---
5. Master and signal files
5.1 `hcps.json`
Field	Type	Notes
hcp_id	string	PK
market_code	string	FK
first_name, last_name, display_name	string	"Dr. Hanna Vogel"
hcp_type	enum	`Specialist` · `GP` · `NP/PA`
specialty	string	from brand key specialties + GP
years_in_practice	int	2–40
gender	enum	`F` · `M`
region	string	4–6 regions per market
urbanicity	enum	`Urban` · `Suburban` · `Rural`
practice_type	enum	`Hospital` · `Private clinic` · `Group practice` · `Academic`
territory_id	string	
brick_id	string|null	MKT_B only
primary_hco_id	string|null	
rep_user_id	string|null	
kol_flag	bool	~5%
in_customer_master	bool	true for all rows (vendor file tests mismatches)
created_date	date	
Distribution: Specialist 55% / GP 35% / NP/PA 10% (MKT_C: NP/PA 0%). Urbanicity A: 55/30/15, B: 60/25/15, C: 70/15/15. Rural share in Segment A deliberately low for BRV in MKT_A (bias case).
5.2 `accounts.json`
`hco_id, market_code, name, archetype, hco_type (Hospital|Clinic|IDN|Specialty centre|Pharmacy chain), region, urbanicity, beds (int|null), annual_patient_volume_ta (object by brand_code → int), affiliated_hcp_count, kam_user_id, procurement_model (Central tender|Local formulary|Direct purchase), treatment_capability, decision_model`
Names: fictitious ("Westmoor General Hospital", "Larkfield Oncology Centre", …). Pareto: top 20% of accounts hold ~65% of patient volume.
5.3 `affiliations.json`
`hcp_id, hco_id, market_code, affiliation_type (Primary|Secondary), weight (0–1), start_date`
Each HCP: 1 primary (weight 0.6–1.0), 0–2 secondary. Weights per HCP sum to 1.
5.4 `consent.json`
`hcp_id, market_code, channel (Email|Portal|Remote|Events|Face-to-face), consent_status (Granted|Withdrawn|Not captured), updated_date`
Consent granted for Email: A 72%, B 64%, C 18%.
5.5 `sales.json`
Granularity differs by market — this is intentional.
Field	Type	Notes
market_code	string	
granularity	enum	`HCP` (A) · `BRICK` (B) · `ACCOUNT_SELLIN` (C)
entity_id	string	hcp_id / brick_id / hco_id
brand_code	string	
period	string	A,B: `2025-11` … `2026-09` (monthly, 12) · C: `2025-Q4` … `2026-Q3`
trx	number|null	A only
nrx	number|null	A only
nbrx	number|null	A only
units	number	all
value_usd	number	all
market_share	number|null	A, B
Shapes: AUR rising (launch, +6–10% per month in A), ZEN growing, CRD flat, NEU growing, BRV flat/slight decline. HCP-level volume Pareto (top decile ~45% of volume).
5.6 `crm_activity.json`
`activity_id, market_code, hcp_id, hco_id (nullable), brand_code, date, channel (F2F|Remote|Email-rep|Event), outcome (Completed|Declined|No show), samples_given (int), call_plan_flag (bool), rep_user_id`
5.7 `digital_engagement.json`
`market_code, hcp_id, brand_code, month, portal_visits, email_opens, email_clicks, webinar_attended (int), content_minutes`
Only HCPs with consent ≠ Withdrawn. Hero: HCP-B-0001 portal visits 3,3,3,4,5,6 for 2026-04…2026-09.
5.8 `pmr_responses.json`
`response_id, market_code, hcp_id, brand_code, wave (2025-W2|2026-W1|2026-W2), wave_date, attitudinal_segment, behavioural_segment, likelihood_to_prescribe (1–10), unmet_need_score (1–5)`
Coverage: A 35%, B 30%, C 55% of sample HCPs (survey-led relies on PMR).
5.9 `formulary_status.json`
`hco_id, market_code, brand_code, status (Listed|Restricted|Under review|Not listed), effective_date, previous_status, source (Tender|P&T committee|Regional formulary)`
Includes ACC-B-001 ZEN change on 2026-10-08.
5.10 `kol_influence.json`
`hcp_id, market_code, therapy_area, influence_score (0–100), publications_3y, congress_talks_3y, network_degree` — A full, B partial (50% of kol_flag HCPs), C `null` for all except 8 named local experts.
5.11 `rep_notes.json`
`note_id, market_code, hcp_id, hco_id, rep_user_id, date, brand_code, text, ai_extracted (bool), extracted_signals [{dimension_code, direction (Up|Down|None), proposed_value, evidence_quote, confidence}]`
~300 notes; 40 with `ai_extracted: true`; includes NOTE-000001 and a conflicting note for HCP-B-0007 ("Trial driven by samples; unlikely to continue").
5.12 `feature_store.json`
`feature_id, feature_name, description, source_category (Survey|CRM|Sales|Claims & access|Digital|Reference), granularity, refresh_frequency, availability {MKT_A, MKT_B, MKT_C: Available|Partial|Not available}, data_age_days {MKT_A, MKT_B, MKT_C: int|null}, used_by_dimensions []`
~25 features, e.g. `trx_qoq_change`, `brick_sales_qoq_change`, `portal_visits_3m`, `call_acceptance_rate`, `pmr_attitudinal`, `formulary_status`, `kol_influence_score`, `rep_adoption_assessment`.
---
6. Segmentation files
6.1 `segment_assignments.json` (current CRM values)
`market_code, customer_type (HCP|HCO), customer_id, brand_code, indication_code (nullable), dimension_code, value, set_date, set_by (Model|Back office|Rep|Agency file|KAM), source_version_id, confidence (High|Medium|Low), gap_filled (bool)`
One row per customer × brand × dimension. Segment age = `2026-10-15 − set_date`.
Segment age distribution (median): A 70 days · B 210 days · C 330 days. % stale (>180d): A 12% · B 58% · C 81%.
Segment mix (HCP, AUR): A 8/17/30/25/20 % (A–E) · B 12/22/28/20/18 · C 20/30/25/15/10 (agency inflates top segments — shows non-comparability).
`gap_filled = true`: C 34% of POTENTIAL values, B 9%, A 0%.
6.2 `segment_history.json`
`history_id, customer_id, customer_type, brand_code, dimension_code, value, valid_from, valid_to (nullable), change_reason (Annual refresh|Bulk refresh|Approved proposal|Override|Agency file), approver_user_id (nullable)`
Gives D01 timeline; include 3 historic informal overrides with no approver (pre-Continuum) in MKT_B.
6.3 `segment_library.json`
`version_id, name, market_code, brand_code, customer_type, method (KMEANS|RULES|RULES_TEMPLATE|HYBRID), parameters {k, features[], weights{}, filters{}}, status (Draft|Approved|Active|Retired), author_user_id, created_date, approved_by, approved_date, activated_date, records_segmented, mapped_to_global (bool), notes`
Seed: VER-0001 MKT_A AUR KMEANS Active · VER-0002 MKT_A AUR KMEANS Retired · VER-0003 MKT_B AUR RULES Active · VER-0004 MKT_C AUR "Agency file 2025" Active, mapped_to_global false · VER-0005 MKT_C AUR RULES_TEMPLATE Draft (created in demo) · plus ~8 others across brands.
6.4 `studio_outputs.json`
Per version: `version_id, cluster_id, cluster_size, centroid {feature: value}, feature_importance [{feature, importance}], box_stats [{feature, min, q1, median, q3, max}], suggested_name, suggested_global_value, before_after_counts (for Sankey: [{from, to, count}])`
MKT_A K-means k=5 with clearly separable clusters; feature importance led by `trx_qoq_change`, `portal_visits_3m`, `pmr_attitudinal`.
---
7. Change and record files
7.1 `change_events.json` (Stage 1 · Signal watch)
`event_id, market_code, customer_type, customer_id, brand_code, signal_type (Sales trend|Rx trend|Digital engagement|PMR wave|Rep note|Formulary change|CRM activity|Affiliation change), source_category, detected_by (Agent|GenAI), detected_at, magnitude, direction (Up|Down), description, data_age_days, led_to_drift_flag (bool)`
~40% of events lead to a drift flag. Includes HCP-B-0010 event with `led_to_drift_flag: false`.
7.2 `drift_flags.json` (Stage 2 · Drift detection)
`flag_id, market_code, customer_type, customer_id, brand_code, dimension_code, current_value, indicated_value, drift_score (0–1), confidence, threshold_ref, supporting_event_ids [], independent_signal_count, model_version, flagged_at`
7.3 `proposals.json` (Stage 3–4 · Explanation + routing)
Field	Type	Notes
proposal_id	string	
market_code, customer_type, customer_id, brand_code, indication_code		
dimension_code	string	
current_value, proposed_value	string	
segment_age_days	int	at proposal time
confidence	enum	
drivers	object[]	`{label, value, source_category, data_age_days}`

flag_ids	string[]	
origin	enum	`Event-driven` · `Bulk refresh` · `Propagation`
origin_ref	string|null	e.g. formulary event or version_id
approver_role	enum	from approval_rights
approver_user_id	string	
status	enum	`Proposed` · `Held` · `Approved` · `Rejected` · `Written back`
hold_reason	string|null	`Conflicting signals` · `Freeze window` · `Change-frequency limit` · `Insufficient evidence`
policy_rule_ref	string|null	
explanation_key	string	key into `ai_cache.json` (AI-3)
created_at, decided_at	timestamp|null	
decided_by, decision_reason	string|null	
Seed status mix (rows): Proposed 45% · Held 15% · Approved 15% · Rejected 10% · Written back 15%. All hero proposals start `Proposed` or `Held` as specified in §3.
7.4 `audit_log.json` (Stage 5 · Record)
`audit_id, timestamp, market_code, actor_user_id, actor_persona, action (Proposal approved|Proposal rejected|Proposal held|Written back|Version approved|Version activated|Threshold changed|Mapping accepted|Recalibration approved), entity_type, entity_id, before_value, after_value, reason, evidence_refs []`
Seed ~400 entries over last 120 days. Invariant: every `Written back` has a preceding `Proposal approved` with a named approver.
7.5 `publish_log.json`
`publish_id, market_code, version_id, published_at, published_by, records_written, target_systems ["CRM segment field","Proposed-segment field","Campaign audiences","NBA context"], status (Success|Partial)`
7.6 `downstream_impact.json`
`market_code, brand_code, consumer (Targeting|Call planning|Next-best-action|Journeys|Segment-health dashboard), metric, before, after, period`
Examples: Call planning — A/B segment calls planned 1,840 → 2,060; Targeting — target list size 610 → 642; NBA context — records refreshed 0 → 186; Journeys — audience re-entries 74.
7.7 `recalibration.json`
`recommendation_id, market_code, dimension_code, issue, evidence {rejections, rejection_rate, top_reason, override_rate_before, override_rate_after, coverage_of_rising_customers}, recommendation, status (Recommended|Under validation|Approved|Declined), model_owner_user_id`
Seed 3: e.g. MKT_A ADOPTION — 38% of rejections "Temporary behaviour" → raise lookback 90→120 days; status Recommended.
---
8. Intake and AI files
8.1 `vendor_file_mkt_c.json` (+ `.xlsx`)
Raw agency columns: `Ref_No, Dr_Name, Spec, Clinic, City, Pot_Class, Segmen, Adopt, Last_Visit, Remarks`. Issues per §3.5. The `.xlsx` is identical and used for upload.
8.2 `ai_cache.json` (object)
```json
{
  "AI-1": { "vendor_mkt_c_v1": { "mappings": [...], "value_maps": {...}, "confidence": {...} } },
  "AI-2": { "VER-0001": [ { "cluster_id": "cluster_2", "name": "...", "description": "...", "global_value": "..." } ] },
  "AI-3": { "PRP-000001": { "headline": "...", "drivers": [...], "confidence": "High",
                             "data_age": "...", "what_would_move_next": "..." } },
  "AI-4": { "segments older than 9 months": { "answer": "...", "table": [...], "link": "S12" } },
  "AI-5": { "NOTE-000001": { "signals": [...] } }
}
```
Required coverage: AI-1 full vendor file; AI-2 all clusters of VER-0001 and VER-0005; AI-3 every hero proposal + 30 others; AI-4 ≥ 12 canned questions (list in `PROMPT_PACK.md`); AI-5 all 40 extracted notes.
Text rules: factual, ≤ 60 words per explanation, cites drivers and data age, no promotional or clinical claims.
---
9. `aggregates.json` (full-universe figures)
Object keyed by market (plus `ALL`) and brand. All screen KPIs describing the full universe read from here.
```json
{
  "as_of": "2026-10-15",
  "markets": {
    "MKT_A": {
      "hcp_universe": 8000, "account_universe": 900,
      "segment_age": {"median_days": 70, "pct_stale_180": 0.12, "histogram": [...]},
      "segment_mix": {"AUR": {"A":0.08,"B":0.17,"C":0.30,"D":0.25,"E":0.20}, "...": {}},
      "data_quality": {"hcp_match_rate":0.97,"duplicate_rate":0.01,"pct_primary_affiliation":0.94},
      "pipeline_30d": {"change_events":620,"drift_flags":160,"explanations":160,
                       "proposals":95,"held":18,"approved":58,"rejected":12,"written_back":52},
      "time_to_update_days": {"baseline": 92, "current": 9},
      "override_rate": {"before":0.21,"after":0.08},
      "acceptance_rate": 0.83,
      "approval_cycle_days": 2.4,
      "standardisation": {"on_global_standard": false, "conformance": 0.71}
    },
    "MKT_B": { "...": "pipeline_30d events 430, flags 108, proposals 64, held 15, written back 33; median age 210" },
    "MKT_C": { "...": "pipeline_30d events 190, flags 44, proposals 27, held 8, written back 12; median age 330; conformance 0.42" },
    "ALL":   { "pipeline_30d": {"change_events":1240,"drift_flags":312,"explanations":312,
                                "proposals":186,"held":41,"approved":97,"written_back":97} }
  },
  "value_levers": { "...": "inputs per §10.2" }
}
```
Market totals must sum to `ALL`.
---
10. Metric definitions
10.1 Dashboard metrics
Metric	Formula	Source
Segment age (days)	today − `set_date`	segment_assignments
Median segment age	median over customers × governed dimensions	aggregates / assignments
% stale	share with age > 180 days	same
HCP match rate	matched rows / total rows	vendor file, aggregates
Duplicate rate	duplicate IDs / total	hcps, vendor file
Conformance	values in global vocabulary / total values	assignments + vocabulary_map
Data-quality score (0–100)	40×match rate + 30×conformance + 20×completeness + 10×(1−duplicate rate)	intake
Data age (days)	today − last refresh of source	feature_store
Drift rate	drift flags / customers in scope	drift_flags
Acceptance rate	approved / (approved + rejected)	proposals
Approval cycle time	mean(`decided_at` − `created_at`)	proposals
Backlog	Proposed + Held	proposals
Time to update	write-back date − first supporting change event date	proposals + events
Override rate	manual changes without proposal / all changes	segment_history
Upgrade / downgrade rate	moved up / down one or more levels ÷ total	studio_outputs / history
Stability index	1 − (customers changed ÷ customers) per period	history
Opportunity gap	HCPs with Potential 3–4 and Adoption ≤ Consideration	assignments
Accessible potential	Σ account potential where ACCESS_STATUS ∈ {Listed, Restricted}	accounts + formulary
Call-plan adherence	completed planned calls ÷ planned calls, by segment	crm_activity
Effort on declining segments	share of calls to customers whose proposal is a downgrade	crm_activity + proposals
Representation index	segment share of group ÷ universe share of group (1.0 = parity; alert < 0.8 or > 1.25)	assignments + hcps
Unapproved write-backs	`Written back` without prior `Proposal approved` (must be 0)	audit_log
10.2 Value calculator (four POV levers)
Lever	Formula	Default inputs (editable)
Earlier coverage of rising opportunity	upgraded customers × weeks detected earlier × value per added covered week	420 HCPs × 9 weeks × $180
Less effort on declining customers	calls on downgraded customers × cost per interaction (redeployed)	3,100 calls × $145
Higher field acceptance	(override rate before − after) × planned calls × value per executed call	(0.21−0.08) × 48,000 × $60
Lower segmentation maintenance	(manual hours per refresh × cycles) before − after × hourly cost	Σ markets: (120×4 + 260×1 + 340×1) − 30% of that, × $85
Show each lever, total, and a note: "Illustrative on synthetic data — populated with your baseline in a pilot."
---
11. Validation rules (must pass before data is shipped)
Every FK resolves: `hcp_id`, `hco_id`, `market_code`, `brand_code`, `version_id`, `flag_id`, `event_id`, `user_id`.
All enum values are in this spec; all segment values are in `dimensions.json.allowed_values`.
MKT_C has no `sales.json` rows with `granularity = HCP` or `BRICK`, and no non-null `trx`.
Brands NEU and BRV have no rows in MKT_C.
No date after 2026-10-15 except freeze windows and future scheduled refreshes.
Every hero record in §3 exists with the exact specified values.
`aggregates.json` market figures sum to `ALL`.
Audit invariant: 0 unapproved write-backs.
Every proposal has ≥ 1 driver and an `explanation_key` present in `ai_cache.json`.
Held proposals all have `hold_reason` and `policy_rule_ref`.
Distributions look realistic: no perfectly uniform splits; Pareto shapes on volume; noise on time series.
Names are fictitious; no real hospitals, products or people.
---
12. Future database mapping
JSON file	Table	Primary key
hcps.json	`hcp`	hcp_id
accounts.json	`hco`	hco_id
affiliations.json	`affiliation`	(hcp_id, hco_id)
segment_assignments.json	`segment_assignment`	(customer_id, brand_code, dimension_code)
proposals.json	`proposal`	proposal_id
audit_log.json	`audit_log`	audit_id
(all others)	same name, singular	their `_id` field
Only `dataService.ts` changes when moving to a database or API.
