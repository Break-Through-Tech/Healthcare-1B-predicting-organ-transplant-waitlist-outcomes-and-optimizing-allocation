# data/ — DUA-governed extracts (LOCAL ONLY)

These CSVs are **row-level** OPTN registry extracts governed by a Data Use
Agreement. Do **not** commit, upload, or share them outside the team.

| folder | cohort | rows | notes |
|---|---|---|---|
| `extract_2015_kidney_only/` | kidney-alone, listed 2015+ (`WL_ORG==KI`) | 494,862 | **primary** — used by analysis + app |
| `extract_2015_ki_kp/` | kidney + kidney-pancreas, 2015+ | 511,390 | includes KP combos |
| `extract_2010_ki_kp/` | kidney + kidney-pancreas, 2010+ | 701,496 | widest window |

Each folder has: `kidney_waitlist_analytic.csv`, `extract_summary.txt`,
`column_manifest.csv`.

Rebuild any of these from the raw STAR file with `extraction/build_analytic_extract.py`.

## Obtaining the raw STAR file

The extracts are built from the OPTN **STAR** (Standard Transplant Analysis and
Research) files — specifically `KIDPAN_DATA.DAT` plus its dictionary
`KIDPAN_DATA.htm`. These are **not** in this repo and are **not** a direct
download.

**Access process**
1. STAR data are released by OPTN/UNOS (under contract to HRSA) **by request
   only**, and require a **signed Data Use Agreement (DUA)**.
2. For this project, the **Challenge Advisor** submitted the data request and
   manages the DUA — individual team members do **not** file their own request.
   Access terms are set by OPTN; the advisor confirms the permitted handling
   arrangement before any data is shared with the team.
3. Once received, the delivery is a set of delimited-text folders (one per organ
   family). We use only the **Kidney / Pancreas / Kidney-Pancreas** folder.
4. Request/status reference: OPTN data — https://optn.transplant.hrsa.gov/data/

**Where it lives locally & how to point the pipeline at it**

The raw file is kept **outside this repo** (it is large — ~1.4 GB — and
DUA-governed). On the original machine it lives at:

```
~/Downloads/Delimited Text File 202606/Kidney_ Pancreas_ Kidney-Pancreas/KIDPAN_DATA.DAT
```

Build the primary kidney-alone extract by pointing the script at your copy:

```bash
~/ckd-ml/bin/python extraction/build_analytic_extract.py \
    --input "/path/to/KIDPAN_DATA.DAT" \
    --outdir data/extract_2015_kidney_only \
    --min-year 2015
```

Keep `KIDPAN_DATA.htm` in the **same folder** as the `.DAT` — the script reads
column names from it (the `.DAT` has no header row). Handle the raw file under
the same DUA terms as the extracts: never commit, upload, or share it.
