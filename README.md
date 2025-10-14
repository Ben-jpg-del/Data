# Mintlify Competitor Data/Competitor Signal MAS

## Purpose

Derive competitor hiring signals from public profiles and auto-compose an exec-ready memo.

## Pipeline

**HTTP Ingest → Normalize (Code) → Split in Batches → Orchestrator (LLM, T=0) → Workers A/B/C/F/G/J → Aggregate → Summarizer (LLM)**

## Agent Protocol (Fixed Order)

* **A — Funding Timeline:** Normalize rounds `{seed,a,b,c,later}` with `{date_utc, amount_usd, investors[]}` via search/news HTTP.
* **B — Talent↔Funding Alignment:** Compute `first_start_utc`, `delta_days_to_nearest_round`, `hired_within_90d_of_round`.
* **C — Role/Seniority:** `role_bucket ∈ {ENG,DATA,DESIGN,GTM,OPS,PEOPLE,PRODUCT}`, `seniority ∈ {JR,MID,SR,LEAD,DIRECTOR+}`.
* **F — Skills Trend:** Recent-weighted top-k skills with `recent_skill` flags.
* **G — Feeder Company:** Immediate previous employer.
* **J — Velocity Flags:** Booleans for starts in last 30/60/90 days.

**Execution semantics:** Orchestrator carries ephemeral per-person state; workers are pure `input → single-line JSON`. Invalid JSON → field `null` + list in `data_quality.missing`. Deterministic via **T=0** and fixed call graph.

## I/O Contract (Per-Person)

```json
{
  "company_key": "string",
  "person_id": "string",
  "linkedinUrl": "string|null",
  "role_bucket": "ENG|DATA|DESIGN|GTM|OPS|PEOPLE|PRODUCT|null",
  "seniority": "JR|MID|SR|LEAD|DIRECTOR+|null",
  "funding_timeline_used": { "seed"?:{}, "a"?:{}, "b"?:{}, "c"?:{}, "later"?:{} } | null,
  "first_start_utc": "ISO8601|null",
  "delta_days_to_nearest_round": 0,
  "hired_within_90d_of_round": true,
  "skills_recent": [{ "name": "string", "recent_skill": true }],
  "feeder_company": { "name": "string", "companyLinkedinUrl"?: "string" } | null,
  "velocity_flags": { "start_in_last_30d": false, "start_in_last_60d": false, "start_in_last_90d": true },
  "data_quality": { "missing": ["field", "..."] }
}
```

## Architecture
<img width="1172" height="533" alt="image" src="https://github.com/user-attachments/assets/ffa99259-14cc-4930-9f0c-556ddf269e76" />

