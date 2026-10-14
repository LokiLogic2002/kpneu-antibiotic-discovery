# AI-driven antibiotic discovery against *Klebsiella pneumoniae*

MSc project · work in progress · Days 1–13 of a 20-day plan completed

## Overview

*Klebsiella pneumoniae* is a bacterium that has become resistant to many antibiotics. This project asks one question: **can a computer learn what known antibiotics look like, invent new molecules in the same spirit, and narrow them down to a short list worth testing in the lab?**

```
ChEMBL MIC data → clean & label → QSAR model ("judge") ─────────────┐
                                                                     ├→ score → cluster → candidates
known actives → REINVENT 4 generator → filter unsuitable molecules ──┘
```

1. **Data:** published ChEMBL lab results (MIC, the lowest dose that stops growth) are cleaned and labelled active or inactive.
2. **Judge:** a QSAR model scores how likely a molecule is to be an antibiotic, with a check on where the score can be trusted.
3. **Generator:** REINVENT 4, a chemical language model, is fine-tuned on the best known actives and invents new molecules.
4. **Filter, score, cluster:** unsuitable molecules are removed, the rest are scored by the judge and grouped by similarity.

## Status

| Stage | Days | Status |
|---|---|---|
| Data download, cleaning, labelling | 1–3 | Done |
| Split, baseline models, applicability domain | 4–6 | Done |
| Molecule generator | 7–10 | Done |
| Optional reinforcement learning | 11 | Buffer day, not done |
| Filtering, scoring, clustering | 12–13 | Done |
| Final candidate selection, 50-structure review grid, `candidates_v1.csv` | 14 | **Postponed** by supervisor: data to be revisited and cleaned first |

## Results at a glance

| Stage | Key result |
|---|---|
| **Data** | 26,942 compounds → 15,076 unique → 14,208 labelled; 5,933 active (41.8%). Active = MIC ≤ 16 µg/mL. |
| **Split** | Scaffold split (molecules sharing a core shape stay together): train 11,366 / validation 1,420 / test 1,422. |
| **Model** | Random Forest on Morgan fingerprints. Scaffold test ROC-AUC **0.867**, PR-AUC 0.842, MCC 0.556. |
| **Trust check** | A molecule is flagged if its closest training molecule has similarity < 0.4. Test ROC-AUC is 0.887 inside this zone and 0.553 outside. |
| **Generator** | Fine-tuned on 3,362 actives (MIC ≤ 8), epoch 8. At temperature 1.0: 94.3% valid, 83.1% unique, 71.2% novel. |
| **Filtering** | 39,201 generated molecules → **13,203** after removing copies, near-copies, unsuitable size or flexibility, and PAINS and Brenk alerts. |
| **Scoring** | All 13,203 scored; top **10,000** kept. 27.7% of scored molecules are outside the trust zone (flagged, not removed). |
| **Clustering** | Top 10,000 grouped at three strictness levels: 3,488 / **2,087** / 979 clusters (cutoffs 0.5 / 0.6 / 0.7). |

### Filtering funnel (Days 12–13)

| Step | Remaining | Removed |
|---|---|---|
| Valid unique generated molecules | 39,201 | – |
| Exact matches to known training actives | 37,398 | 1,803 |
| Near-duplicates (similarity ≥ 0.8) | 33,439 | 3,959 |
| Molecular weight 150–700 | 31,711 | 1,728 |
| Rotatable bonds ≤ 12 | 30,055 | 1,656 |
| PAINS alerts | 28,293 | 1,762 |
| Brenk alerts | 13,203 | 15,090 |
| Top 10,000 by QSAR score | 10,000 | 3,203 |

## Notebooks (run in this order)

| Notebook | Day | Purpose |
|---|---|---|
| `00_environment_check` | 1 | Check GPU, Python and tools |
| `00b_chembl_download` | 2 | Download raw ChEMBL data |
| `02_data_cleaning` | 2 | Standardise compounds, flag conflicts and suspect values |
| `03_activity_labels` | 3 | Label compounds active / inactive |
| `04_splits` | 4 | Scaffold-based train / validation / test split |
| `05_baseline_models` | 5 | Compare Random Forest, logistic regression, XGBoost |
| `06_applicability_domain` | 6 | Define the trust zone; save the final model |
| `07_generator` | 7–10 | REINVENT 4 fine-tuning, epoch choice, temperature sweep |
| `12_filter_score_cluster` | 12 | Filter generated molecules |
| `13_score_cluster` | 13 | Score, annotate, keep top 10,000, cluster |

A day-by-day record with settings, outputs and decisions is kept in the project progress sheet.

## Decisions to confirm with supervisor

| # | Decision | Current choice | Why it needs a check |
|---|---|---|---|
| 1 | Fine-tuning epoch | 8 | Lowest validation loss, but epoch 5 copies training molecules less (6.8% vs 10.5%) |
| 2 | Sampling temperature | 1.0 | 1.2 gives more novelty but only 89.7% valid, just under the ~90% bar |
| 3 | Novelty reference | All 4,180 known train actives | Includes borderline compounds (MIC > 8–16) that were excluded from fine-tuning |
| 4 | Near-duplicate cutoff | 0.8 | Plan allows 0.7–0.8 |
| 5 | Rotatable-bond limit | ≤ 12 | Some known actives exceed 10 |
| 6 | Brenk filter | Applied | Removed about half of the molecules, and many known actives trigger Brenk alerts |
| 7 | Molecules kept after scoring | 10,000 | More variety than 5,000, but 14.1% are outside the trust zone |
| 8 | Clustering cutoff | 0.6 (0.5 as backup) | 0.7 puts almost half the molecules in one cluster |
| 9 | Scope of "clean up the data" | – | Needed before Day 14 |

## Known limitations

- Model scores follow similarity to known antibiotics closely, so the ranking favours familiar-looking molecules.
- Scores outside the trust zone are less reliable. This affects 14.1% of the top 10,000, and about 2,050 of them score below 0.5.
- 3D shape values come from one generated shape per molecule and are rough guides only (90 molecules failed).
- The primary-amine check may miss some charged amines.
- Annotations (amine, rigidity, shape) are labels only; nothing was filtered on them.

## Reproducing the work

Notebooks run on Google Colab (T4 GPU) with data in Google Drive. Run them in order. The generator notebook installs REINVENT 4; filtering and scoring are in separate notebooks because the QSAR model needs scikit-learn 1.6.1 and the REINVENT install changes package versions.

Large files (fingerprints, trained models, model checkpoints, raw downloads) are not stored in this repository because of GitHub size limits; they are regenerated by the notebooks.

## Author

[SMARANIKA RANA] · MSc [BIOTECHNOLOGY], [GARDEN CITY UNIVERSITY] · Supervisor: [PARTH SANGHAVI]
