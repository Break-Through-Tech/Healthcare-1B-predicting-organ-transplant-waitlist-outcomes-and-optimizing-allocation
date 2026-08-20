# Data Dictionary — OPTN Kidney Waitlist Analytic Extract

Project-specific data dictionary for the analytic extract used in this repo.
Source: OPTN `KIDPAN` STAR file (June 2026 release), **kidney-alone** waitlist
registrations listed **2015-01-01 onward**. This is an *expanded, curated*
subset of the 475-variable STAR file — see the official OPTN dictionary
(`KIDPAN_DATA.htm`) for the full variable list.

- **Cohort:** `WL_ORG == "KI"` and `INIT_DATE ≥ 2015-01-01`
- **Rows:** 494,862 registrations · **Columns:** 34 retained + 5 derived
- **Unit of analysis:** one waitlist *registration* (a patient listed at
  multiple centers appears more than once)

> ⚠️ This file documents the data; it contains **no row-level records**. The
> extracts themselves are DUA-governed and are not distributed. Rebuild them
> with `extraction/build_analytic_extract.py`.

---

## Role legend

| Role | Meaning |
|---|---|
| **ID** | Encrypted linkage key; not modeled |
| **cohort** | Used to define / subset the population |
| **feature** | Listing-time model input (one of 11) |
| **outcome** | Used to build the prediction label |
| **leaky** | Known only **at/after transplant** — excluded from models to prevent target leakage |
| **derived** | Computed by our pipeline |
| **descriptive / unused** | Kept for reference, not used as a feature |

`% miss` = share missing in the 494,862-row cohort. A cluster at ~53% marks
transplant-time fields (missingness ≈ the non-transplant rate).

---

## Columns

### Identifiers
| Variable | Definition | Type | Role | % miss |
|---|---|---|---|---:|
| `PT_CODE` | Encrypted patient ID; tracks a person across multiple listings | str | ID | 0.0 |
| `WL_ID_CODE` | Encrypted waitlist-registration ID | str | ID / waitlist filter | 0.0 |
| `TRR_ID_CODE` | Encrypted transplant-event ID | str | ID (transplant-time) | 53.4 |
| `DONOR_ID` | Encrypted donor ID | str | ID (transplant-time) | 53.4 |

### Cohort & timing
| Variable | Definition | Type | Role | % miss |
|---|---|---|---|---:|
| `WL_ORG` | Organ the candidate is **waitlisted** for (KI/KP/PA/PI) | cat | **cohort key** | 0.0 |
| `ORGAN` | Organ actually **transplanted** (null until transplant) | cat | leaky | 53.4 |
| `WLKI` | "Listed for kidney" flag | cat | unused (100% null here) | 100.0 |
| `INIT_DATE` | Date placed on the waiting list | date | cohort window + time origin | 0.0 |
| `END_DATE` | Date the registration ended (removal/transplant/death/cutoff) | date | time-to-event | 0.0 |
| `REM_CD` | Reason the registration ended (removal code) | int | **primary outcome source** | 20.9 |

### Listing-time features (model inputs)
| Variable | Definition | Type | Role | % miss |
|---|---|---|---|---:|
| `INIT_AGE` | Age (years) at listing | num | feature | 0.0 |
| `GENDER` | Sex | cat | feature | 0.0 |
| `ETHCAT` | Race/ethnicity category code | cat | feature | 0.0 |
| `ABO` | Blood group at registration | cat | feature | 0.0 |
| `INIT_CPRA` | Calculated PRA % at listing (sensitization) | num | feature | 28.8 |
| `ON_DIALYSIS` | On dialysis at listing | cat | feature | 0.0 |
| `BMI_TCR` | Body-mass index at registration | num | feature | 0.3 |
| `FUNC_STAT_TCR` | Functional-status code at registration | cat | feature | 0.8 |
| `INIT_STAT` | Initial medical-urgency status code | cat | feature | 0.0 |
| `REGION` | OPTN region (1–11) | cat | feature | 0.0 |

### Transplant-time / end-of-episode (excluded as leaky)
| Variable | Definition | Type | Role | % miss |
|---|---|---|---|---:|
| `END_CPRA` | CPRA at end of episode | num | leaky | 28.8 |
| `END_STAT` | Medical-urgency status at removal | cat | leaky | 0.0 |
| `PREV_TX` | Prior-transplant indicator (populated at transplant here) | cat | leaky | 53.4 |
| `DIAG_KI` | Primary kidney diagnosis code (populated at transplant here) | cat | leaky | 53.7 |
| `TX_DATE` | Transplant date | date | leaky | 53.4 |
| `DON_TY` | Donor type (deceased vs living) | cat | leaky | 53.4 |
| `PTIME` | Patient survival time (days) | num | leaky | 54.0 |
| `PSTATUS` | Patient status (1=dead, 0=alive) | int | leaky | 53.4 |
| `COMPOSITE_DEATH_DATE` | Best-available death date | date | leaky | 80.5 |

### Descriptive / unused
| Variable | Definition | Type | Role | % miss |
|---|---|---|---|---:|
| `DAYSWAIT_CHRON` | Total days on list incl. inactive time | num | descriptive | 0.0 |
| `DAYSWAIT_ALLOC` | Days counted toward allocation priority | num | descriptive | 1.4 |
| `DIALYSIS_DATE` | Dialysis start date | date | descriptive | 27.1 |
| `LISTING_CTR_CODE` | Listing-center code | str | descriptive | 0.0 |
| `MULTIORG` | Multi-organ listing flag | cat | unused (98% null) | 97.6 |
| `A2A2B_ELIGIBILITY` | A2/A2B→B eligibility flag | cat | unused (92% null) | 92.3 |

### Derived fields (added by our pipeline)
| Variable | Definition | Type | Role |
|---|---|---|---|
| `outcome` | Coarse bucket from `REM_CD` (see mapping below) | cat | label source |
| `event_adverse` | 1 if `died` or `removed_too_sick`, else 0 | int | **classification target** |
| `event_transplant` | 1 if `transplanted`, else 0 | int | **survival event** |
| `censored` | 1 if `still_waiting` at window end | int | survival censoring |
| `days_to_event` | `END_DATE − INIT_DATE` (days; negatives nulled) | num | survival duration |

---

## Code / value mappings

**`GENDER`** — `M`, `F`

**`ETHCAT`** (race/ethnicity) — `1` White · `2` Black · `4` Hispanic · `5` Asian · `6` American Indian · `7` Pacific Islander · `9` Multiracial · `998` Unknown

**`ABO`** (blood group) — `O`, `A`, `B`, `AB`, plus A subtypes `A1`, `A2`, `A1B`, `A2B`

**`ON_DIALYSIS`** — `Y`, `N`

**`WL_ORG`** — `KI` kidney-alone (our cohort) · `KP` kidney-pancreas · `PA` pancreas · `PI` islet

**`REGION`** — OPTN regions `1`–`11` (geographic allocation units)

**`FUNC_STAT_TCR`** (functional status at registration, Karnofsky-style; higher = better function) — adult codes `2010`–`2100` (≈10%–100% performance), alternate scale `4010`–`4100`; `996`/`998` not applicable/unknown

**`INIT_STAT`** (initial medical-urgency status) — kidney status codes, e.g. `4010`, `4020`, `4030`, `4050`, `4060`, `4099` (active)

**`REM_CD` → `outcome` bucket**
| Bucket | `REM_CD` codes | Meaning |
|---|---|---|
| `transplanted` | 2, 4, 15, 18, 19, 41–45 | received a transplant (this registration) |
| `died` | 8, 21, 23 | died on the waiting list |
| `removed_too_sick` | 5, 13 | removed as too sick to transplant |
| `transplanted_elsewhere` | 3, 14, 22 | transplanted at another center/registration |
| `removed_administrative` | 7, 9, 10, 11, 16, 17, 20, 24, 40 | administrative / non-clinical removal |
| `still_waiting` | *missing `REM_CD`* | right-censored at window end |
| `unknown` | any code not listed above | unmapped |

---

## Outcome distribution (n = 494,862)

| outcome | n | share |
|---|---:|---:|
| transplanted | 234,429 | 47.4% |
| still_waiting | 103,300 | 20.9% |
| removed_administrative | 45,332 | 9.2% |
| transplanted_elsewhere | 37,243 | 7.5% |
| removed_too_sick | 35,219 | 7.1% |
| died | 34,524 | 7.0% |
| unknown | 4,815 | 1.0% |
| **adverse (died + too-sick)** | **69,743** | **14.1%** |

_Consistency check: `still_waiting` (20.9%) ≈ `REM_CD` missingness (20.9%)._

---

## Notes on cleaning & leakage

- **No header row.** The `.DAT` has no column names; they are attached from the
  sibling `KIDPAN_DATA.htm` (475 ordered variables) and the file is read under
  `latin-1`.
- **Missing sentinel.** STAR encodes missing as `"."`; mapped to `NA` on ingest.
- **Cohort field.** Filter on `WL_ORG` (waitlisted organ), **never** `ORGAN`
  (transplanted organ) — the latter is null until transplant and collapses the
  cohort to recipients.
- **Leakage rule.** Any variable populated only at/after transplant (the ~53%
  and end-of-episode fields above) is excluded from model features. Only the 11
  listing-time features feed the models.
- **Negative durations.** 59 rows had `END_DATE < INIT_DATE` (data-entry
  errors); their `days_to_event` was set to missing.

## Cohort funnel

| Stage | Rows |
|---|---:|
| Raw `KIDPAN` registrations | 1,303,788 |
| After kidney-alone + 2015+ waitlist subset | **494,862** |

---

*Maintained alongside `analysis/PAPER.md` (§3). Regenerate the source stats with
`extraction/build_analytic_extract.py` (writes `extract_summary.txt` and
`column_manifest.csv` per extract).*
