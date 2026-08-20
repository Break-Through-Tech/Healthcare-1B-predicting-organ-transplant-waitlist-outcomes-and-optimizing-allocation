# Predicting Organ Transplant Waitlist Outcomes & Optimizing Allocation

**Company / Org:** MediMate Foundation
**Challenge Advisor:** Deepti Bahel, baheldeepti@gmail.com
**Program:** Break Through Tech AI Studio — Fall 2026

---

## 🏢 About MediMate Foundation

MediMate Foundation is a California nonprofit building AI-powered navigation tools for patients and caregivers in the kidney and transplant space. Grounded in lived patient experience, our work focuses on transparency, equity, and risk assessment across the organ allocation process — helping patients, clinicians, and the broader community understand who is at risk while waiting and why.

---

## 🎯 The Challenge

### Project Summary

This project builds predictive models and analytical tools to identify risk and improve transparency in organ allocation. The goal is to surface which waitlist patients are at highest risk, estimate expected wait times, and quantify the geographic and systemic factors that influence transplant outcomes — in a way that is fair across patient subgroups and explainable to non-technical stakeholders.

### Success Criteria

Model performance targets: **ROC-AUC ≥ 0.78**, **PR-AUC ≥ 0.65**, **Brier score ≤ 0.18**, **lift in top decile ≥ 2.5×**, and **C-index ≥ 0.72**.

Success is also measured by **fairness** (subgroup performance within 0.05 AUC across available demographic and regional fields) and by the quality of the final deliverables (model, explainability analysis, and stakeholder-facing write-up/dashboard).

### Project Milestones

Use these milestones to guide your work. Your team will create a **GitHub Projects board** to track tasks within each milestone.

| Month | Milestone | Key Activities |
|---|---|---|
| **September** | Data Understanding | Explore dataset, handle missing values, document findings |
| **October** | Model Development | Train baseline model, experiment with approaches, iterate |
| **November** | Evaluation & Presentation | Finalize model, prepare presentation, document results |

> **Note for the team:** Create a GitHub Projects board in this repository to break these milestones into weekly tasks. Go to the **Projects** tab → **New project** → choose **Board** → add a column for each month.

---

## 📊 Dataset

This project works with **publicly available, openly downloadable data** — no data-use agreement, application, or wait. The working dataset is real, patient-level transplant waitlist data; U.S. national registry resources are used only as public context.

### Working dataset (public, instant download)

**Name & Source:** **Kidney Transplant Waitlist (Brazil / SP-OAS)** — a de-identified, patient-level kidney waitlist dataset from the São Paulo State Organ Allocation System, published openly on Kaggle.

- **Download:** https://www.kaggle.com/datasets/gustavomodelli/waitlist-kidney-brazil
- **Size:** ~54,000 candidate records (Jan 2000 – Dec 2017) — large enough to be real, small enough to run on Google Colab / free tiers. **Public, no DUA, no fee, no wait.**
- **Why it fits:** It's a true waitlist-outcomes dataset with exactly the structure this project needs — candidate features at registration (age, sex, blood group, panel-reactive antibody/PRA, subregion, dialysis time) and time-to-event outcomes (deceased-donor transplant vs. death/removal vs. still waiting). It supports classification **and** survival modeling (Cox PH → C-index), so the full metric suite and the fairness/subgroup check all apply.
- **Provenance:** It's the dataset behind a peer-reviewed ML wait-time study (PLOS One, 2021 — Cox proportional hazards, ~54k records), so the team has a published reference pipeline to benchmark against. A worked Kaggle notebook also exists.
- **Important caveat:** This is a Brazilian state system, not the U.S. The **methods transfer directly**, but allocation rules differ from OPTN. Treat conclusions as method demonstrations, **not U.S. policy claims**.

> *Optional additional starter sets to explore on Kaggle (verify license/provenance before relying on them): `fkshaikh/organ-transplant-dataset`, `subhrajyotinath/organdonation`.*

### U.S. context (background reading only — not a work item)

The OPTN **STAR files** and **SRTR** reports are the U.S. national registry resources. They are **out of scope for this semester** (STAR requires a signed DUA with up to a 30-day turnaround). Use the public aggregate reports below purely as context for framing your findings.

- OPTN national data: https://optn.transplant.hrsa.gov/data/
- SRTR: https://www.srtr.org/

---

## ⚠️ Known Challenges & Considerations

- **Class imbalance:** Adverse outcomes (waitlist mortality/removal) are rarer than transplants. Lead with **PR-AUC and calibration (Brier)**, not ROC-AUC alone.
- **Missingness:** Document missingness patterns and pick a principled imputation strategy; don't silently drop rows. (The published study excluded a small number of records missing subregion — note any exclusions you make.)
- **Competing risks:** Transplant, death, and removal are competing outcomes. A plain Cox model can bias wait-time estimates; consider a competing-risks framing and compare.
- **Fairness & leakage:** Use the available demographic/regional fields (age, sex, blood group, subregion) for the subgroup-fairness check. Watch for proxy leakage — don't let the model simply re-learn historical allocation patterns.
- **Distribution shift:** The data spans ~18 years and allocation practice changes over that window. Consider encoding registration era or restricting the window, and check stability over time.
- **Data handling:** Even though this dataset is public, keep raw row-level CSVs **out of this repo** (`.gitignore` them) and commit only code, aggregates, and figures.

---

## 🧭 Suggested Approach

1. **EDA & data dictionary** — profile every field, document distributions, missingness, and outcome base rates. Produce a shared data dictionary the whole team works from.
2. **Baseline classification** — predict adverse waitlist outcome. Start with logistic regression, then tree ensembles (random forest, gradient boosting).
3. **Survival modeling** — Cox proportional hazards for time-to-transplant; evaluate with C-index. Compare against a competing-risks formulation.
4. **Calibration & threshold selection** — Brier score, calibration curves, and decile lift. Pick an operating point tied to a stated use case.
5. **Explainability** — SHAP global and local explanations. Translate the top drivers into plain language a patient or clinician would understand.
6. **Fairness audit** — subgroup performance across age, sex, blood group, and subregion. Report gaps honestly, including where you fall short of the 0.05 AUC target.
7. **Deliverable** — stakeholder write-up plus a lightweight dashboard or notebook walkthrough.

### Suggested sub-teams (6–7 fellows)

EDA/data dictionary · baseline modeling · survival & competing risks · explainability (SHAP) · fairness audit · dashboard & presentation

---

## 🛠️ Getting Started

```bash
git clone <this-repo>
cd <this-repo>
pip install -r requirements.txt
```

Suggested stack: `pandas`, `scikit-learn`, `lifelines` (Cox PH), `scikit-survival`, `shap`, `matplotlib`/`seaborn`, `streamlit` (optional dashboard). Everything runs on Google Colab free tier.

Download the dataset from the Kaggle link above into a local `data/` directory. **`data/` is gitignored — do not commit raw records.**

---

## 📚 Resources

- Kaggle dataset: https://www.kaggle.com/datasets/gustavomodelli/waitlist-kidney-brazil
- OPTN national data (context): https://optn.transplant.hrsa.gov/data/
- SRTR (context): https://www.srtr.org/
- `lifelines` docs (survival analysis): https://lifelines.readthedocs.io/
- `scikit-survival` docs: https://scikit-survival.readthedocs.io/
- SHAP docs: https://shap.readthedocs.io/
