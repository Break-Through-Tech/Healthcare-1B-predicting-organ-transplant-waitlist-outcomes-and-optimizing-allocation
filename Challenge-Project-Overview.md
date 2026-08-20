# Predicting Organ Transplant Waitlist Outcomes & Optimizing Allocation

**Company / Org:** MediMate Foundation  
**Challenge Advisor:** Deepti Bahel, baheldeepti@gmail.com  
**AI Studio Coach:** Nagalakshmi Pulivarthi,nagalakshmi.pulivarthi@breakthroughtech.org   
**Program:** Break Through Tech AI Studio - Fall 2026

---

## 🏢 About MediMate Foundation

MediMate Foundation is a California 501(c)(3) nonprofit dedicated to improving
healthcare outcomes through advanced analytics and predictive modeling. Our focus
is on transparency and risk assessment in the organ allocation process.

We build AI-powered navigation and decision-support tools for people living with
chronic kidney disease and for kidney transplant candidates. Our work is grounded
in lived patient experience — our founder is a kidney transplant recipient — and
everything we publish is aggregate, explainable, and written to be understood by
the people it describes.

---

## 🎯 The Challenge

### Project Summary
This project builds predictive models and analytical tools to identify risk and
improve transparency in organ allocation patterns. The goal is to provide insights
into which patients are at highest risk while waiting, expected wait times, and
geographic or systemic factors influencing transplant success.

Your team will answer three linked questions:

1. **Who is at risk?** Predict which waitlist candidates are most likely to die or
   be removed from the list before receiving a transplant.
2. **How long will they wait?** Model time-to-transplant using survival analysis,
   accounting for candidates still waiting when the observation window ends.
3. **Is the model fair, and can we explain it?** Audit performance across
   demographic and geographic subgroups, and translate the model's drivers into
   plain language.

### Success Criteria
Model performance targets: ROC-AUC >= 0.78, PR-AUC >= 0.65, Brier score <= 0.18,
Lift in top decile >= 2.5x, and C-index >= 0.72. Success is also measured by
fairness (subgroup performance within 0.05 AUC) and deliverable quality.

> **A note on these numbers:** These are targets to aim at, not a pass/fail bar. A
> well-documented model that misses a threshold and explains *why* is a stronger
> result than one that hits it without understanding.

### Project Milestones

Use these milestones to guide your work. Your team will create a **GitHub Projects
board** to track tasks within each milestone.

| Month      | Milestone          | Key Activities                                                  |
|------------|--------------------|----------------------------------------------------------------|
| **September**  | Data Understanding | Explore dataset, handle missing values, document findings       |
| **October**    | Model Development  | Train baseline model, experiment with approaches, iterate      |
| **November**   | Evaluation & Presentation | Finalize model, prepare presentation, document results        |

> **Note for the team:** Please create a GitHub Projects board in this repository
> to break these milestones into weekly tasks. Go to the **Projects** tab →
> **New project** → Choose **Board** → Add columns for each month.

**Suggested sub-teams (6–7 fellows):** EDA & data dictionary · baseline
classification · survival modeling · explainability (SHAP) · fairness audit ·
dashboard & final presentation

---

## 📊 Dataset

**Name and Source:** OPTN national transplant data (Organ Procurement and
Transplantation Network, administered by UNOS under contract to HRSA)  
**Format:** Structured and unstructured data (e.g., STAR files). Delivered as
delimited text, SAS, or Stata.  
**Size:** Multi-table dataset covering transplant recipients and waitlist
candidates back to 10/1/1987 — thousands of records across hundreds of variables.
Exact file size confirmed on delivery. Our working scope is the **kidney** tables
only, which keeps this manageable on Google Colab.  
**Location:** Access is by request, not direct download. See
[`data/README.md`](data/README.md) for the full access process and current status.

### Key Details

- Publicly available OPTN national transplant data, including STAR files with
  structured patient data (blood type, region, age, sex, race/ethnicity, medical
  urgency scores) and unstructured free-text fields for cause of death narratives
  and comorbidity notes.
- **Access requires a signed Data Use Agreement.** The Challenge Advisor has
  submitted the request and is handling the DUA. Students will not need to submit
  their own request — but note that access terms are set by OPTN, and the advisor
  will confirm the permitted handling arrangement before any data is shared with
  the team.
- **Working scope:** Kidney waitlist candidates and transplant recipients,
  2010–present. We are deliberately not using all organs or the full 1987-forward
  history — that scope would spend your whole term on preprocessing.
- **Free-text fields** (cause of death narratives, comorbidity notes) are a
  stretch component, not a core requirement. Get the structured model working
  first.

### Known Limitations and Preprocessing Needed

- **Class imbalance.** Adverse waitlist outcomes are less common than transplants.
  Lead your evaluation with PR-AUC and calibration (Brier score) rather than
  ROC-AUC alone — ROC-AUC can look deceptively good on imbalanced data.
- **Censoring.** Many candidates are still waiting at the end of the observation
  window. Treating "no transplant yet" as a negative label will bias your model.
  This is what survival analysis exists to handle.
- **Competing risks.** Transplant, death, and removal compete — a candidate who
  dies can no longer be transplanted. A standard Cox model can bias wait-time
  estimates; try a competing-risks formulation and compare.
- **Missingness.** Document missingness patterns before choosing an imputation
  strategy, and record any rows you exclude.
- **Distribution shift.** U.S. kidney allocation policy changed substantially in
  2014 (KAS) and again in 2021 (Acuity Circles). A model trained across these
  boundaries may not be stable. Consider encoding policy era and checking
  performance over time — this is one of the more interesting findings available
  in this dataset.
- **Scale.** STAR files require software able to handle large record counts and
  hundreds of variables. Excel will not open these. Use pandas.
- **Fairness and leakage.** Watch for the model simply re-learning historical
  allocation patterns rather than predicting clinical risk. That distinction is
  the heart of the fairness component.

### Data Handling

⚠️ **This is a public repository, and this dataset is governed by a Data Use
Agreement.**

- **Never commit raw row-level records to this repo.** The `data/` folder is
  gitignored for record files.
- Commit code, aggregate summaries, and figures only.
- Do not redistribute the data outside the team.
- If you are unsure whether something is safe to commit, ask before you push.

### Data Dictionary and Documentation

- OPTN STAR file data dictionary is provided by UNOS with the delivered files.
- OPTN data documentation: https://optn.transplant.hrsa.gov/data/
- Building your own expanded, project-specific data dictionary is the first
  September deliverable.

---

## 🛠️ Suggested Approach

**ML Problem Type:** Classification (primary) + Survival Analysis (time-to-event)

> This project is deliberately two-part. Classification answers "who is at risk";
> survival analysis answers "how long until transplant." The C-index in the Success
> Criteria comes from the survival half, so don't skip it.

**Suggested Sequence:**

1. **EDA and data dictionary** — profile every field: distributions, missingness,
   outcome base rates. Produce a shared data dictionary the whole team works from.
2. **Baseline classification** — start with logistic regression as an interpretable
   baseline, then move to tree ensembles (Random Forest, XGBoost). Always report
   the baseline; a complex model that barely beats logistic regression is itself a
   finding worth stating.
3. **Survival modeling** — Cox proportional hazards for time-to-transplant,
   evaluated with C-index. Then compare against a competing-risks model.
4. **Calibration and thresholds** — Brier score, calibration curves, decile lift.
   Choose an operating point tied to a stated use case, and justify it.
5. **Explainability** — SHAP global and local explanations. Then the harder part:
   translate the top drivers into language a patient or clinician would understand.
6. **Fairness audit** — subgroup performance across age, sex, race/ethnicity, blood
   type, and OPTN region. Report gaps honestly, including where you miss the 0.05
   AUC target.
7. **Deliverable** — stakeholder write-up plus a Streamlit dashboard or notebook
   walkthrough.

**Recommended Libraries:**
- XGBoost
- Logistic Regression (scikit-learn)
- Cox Proportional Hazards (Cox PH) — via `lifelines` or `scikit-survival`
- SHAP
- Streamlit
- GitHub
- Jupyter Notebooks
- Docker
- CI/CD
- Large Language Models (LLMs) such as Claude or GPT
- Google Colab

**Evaluation Metrics:**
- ROC-AUC
- PR-AUC
- Brier score
- Lift in top decile
- C-index
- Subgroup AUC gap (fairness)

---

## 📚 Resources to Get Started

The following resources will help your team understand the problem space and
potential technical approaches for this project:

**Background Reading:**
- OPTN — How organ allocation works:
  https://optn.transplant.hrsa.gov/patients/about-transplantation/how-organ-allocation-works/
- OPTN — National data reports: https://optn.transplant.hrsa.gov/data/
- SRTR — Annual data reports on U.S. transplant outcomes: https://www.srtr.org/reports/
- National Kidney Foundation — The transplant waitlist explained:
  https://www.kidney.org/atoz/content/transplant-waitlist

**Technical Tutorials:**
- lifelines — survival analysis in Python:
  https://lifelines.readthedocs.io/en/latest/Survival%20Analysis%20intro.html
- scikit-survival user guide:
  https://scikit-survival.readthedocs.io/en/stable/user_guide/index.html
- SHAP documentation: https://shap.readthedocs.io/en/latest/
- scikit-learn — probability calibration:
  https://scikit-learn.org/stable/modules/calibration.html
- XGBoost — Python introduction:
  https://xgboost.readthedocs.io/en/stable/python/python_intro.html
- Streamlit — get started: https://docs.streamlit.io/get-started

**Code Examples:**
- lifelines Cox PH worked example:
  https://lifelines.readthedocs.io/en/latest/Survival%20Regression.html
- scikit-survival worked examples:
  https://scikit-survival.readthedocs.io/en/stable/user_guide/00-introduction.html

**Other:**
- Fairlearn — fairness assessment in ML: https://fairlearn.org/
- If you find a good paper, notebook, or explainer, post it in Slack — I'd rather
  the team share sources than duplicate searching.

*Feel free to explore beyond these, and share anything interesting you find with me!*

---

## 🤝 How We'll Work Together

**Check-ins:** During our biweekly 60-min AI Studio Lab Section meeting block
(2nd and 4th week of every month)  
**Communication:** Slack (Break Through Tech workspace)  
**Response time:** Within 48 hours on weekdays  

**Recommended Tools:**
- **Coding:** Google Colab, VS Code
- **Collaboration:** GitHub, Notion
- **Virtual Meetings:** Zoom, Google Meet

**What I'm looking for from you:** Honest documentation over polished results. If
something doesn't work, that's data. If a metric target isn't reachable with this
dataset, I want to know why — that's a more interesting finding than hitting the
number.

---

## 🚀 Getting Started

1. **Review this overview document** and note any questions for our first meeting
2. **Begin reviewing the dataset** using the link above — start with
   `data/README.md` for current access status
3. **Read the GitHub Projects documentation**
   [here](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects)
4. **Set up your environment** — a Colab notebook with `pandas`, `scikit-learn`,
   `lifelines`, and `shap` is enough to start

I'm excited to work with you!

---


---
