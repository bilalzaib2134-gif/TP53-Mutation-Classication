# TP53 Variant Pathogenicity Prediction Using Gene-Specific Machine Learning

> **Gene-Specific Prediction of TP53 Variant Pathogenicity Using a Random Forest Model with Sequence k-mer, Positional, and Functional Domain Features**

A gene-specific Random Forest classifier for predicting pathogenicity of TP53 single-nucleotide variants, trained on expert-curated ClinVar data and benchmarked against CADD, ClinPred, SIFT, and PolyPhen-2.

> **Research use only. Not intended for clinical diagnosis.**

---

# Overview

TP53 is altered in approximately half of all human cancers, yet distinguishing pathogenic from benign variants reliably remains a persistent challenge in clinical oncology. Genome-wide pathogenicity predictors such as CADD, SIFT, and PolyPhen-2 are not designed around any single gene's mutational architecture, which can dilute the strong, spatially concentrated signal present in genes like TP53.

This repository contains a gene-specific Random Forest classifier trained exclusively on TP53 variants, integrating local sequence context (k-mer frequencies) with biologically meaningful features: codon position, functional domain membership, hotspot status, genomic location, and nucleotide substitution type.

The complete workflow is fully reproducible using open-source software and Google Colab, and requires no institutional computing infrastructure.

---

# Dataset

Source:

- NCBI ClinVar (April 2026)
- Query: `TP53[gene] AND single nucleotide variant[variant type]`

Downloaded variants:

- Total raw entries: **2,739**
- Retained after VUS/conflicting-classification exclusion: **1,470**
- Excluded VUS/conflicting variants: **1,269** (46.3%)
- Further excluded (SPDI parse/coordinate failure): **1** variant
- **Final analysis dataset: 1,469 variants (410 pathogenic, 1,059 benign)**

Binary labels:

- Pathogenic / Likely Pathogenic → 1
- Benign / Likely Benign → 0

Reference transcript: `NM_000546.6`

---

# Feature Engineering

Total features: **276**

**Sequence features (256)**
- Trinucleotide and tetranucleotide k-mer frequencies
- 101 bp sequence window (±50 bp), centered on the variant position

**Biological features (20)**
- Codon position
- Relative genomic position
- Functional domain membership (7 domains: TAD1, TAD2, Proline, DBD, Linker, Tetramerization, Regulatory)
- Hotspot status (6 canonical codons: R175, G245, R248, R249, R273, R282)
- Reference / alternate nucleotide identity
- Nucleotide substitution type (transition/transversion)

**k-mer vectorizer fitting:** the `CountVectorizer` is fit on training-set sequences only, then applied via `.transform()` to test-set sequences — never fit on the full dataset. This is a corrected step; an earlier development version fit the vectorizer on the full dataset prior to the train/test split, which was identified and fixed prior to this release.

---

# Machine Learning Model

**Classifier:** `RandomForestClassifier` (scikit-learn)

**Parameters:**
- 200 trees
- `class_weight="balanced"`
- `min_samples_split=3`
- `random_state=42`
- `n_jobs=1` (fixed to eliminate parallel non-determinism)

**Evaluation:**
- 80/20 stratified train/test split (1,175 / 294 variants)
- Five-fold stratified cross-validation on the training partition
- Independent held-out test set, evaluated exactly once

---

# Performance

| Metric | Value |
|---|---|
| Test AUC-ROC | **0.9511** (95% CI 0.9221–0.9736) |
| Test AUPRC | **0.8772** |
| Test Accuracy | **0.9048** |
| Five-fold CV AUC | **0.9611 ± 0.0198** |
| Sensitivity | **0.8780** |
| Specificity | **0.9151** |
| PPV | **0.8000** |
| NPV | **0.9510** |
| F1 | **0.8372** |

Held-out test set: **294 TP53 variants** (82 pathogenic, 212 benign)
Confusion matrix: TN = 194, FP = 18, FN = 10, TP = 72

---

# Stratified Performance: DBD vs. Non-DBD

| Region | AUC-ROC | Sensitivity | Specificity | n |
|---|---|---|---|---|
| Overall | 0.9511 | 0.8780 | 0.9151 | 294 |
| DBD (codons 102–292) | 0.7833 | 0.9259 | 0.4000 | 74 |
| Non-DBD | 0.9425 | 0.7857 | 0.9688 | 220 |

Specificity within the DBD is markedly reduced (40.0%) relative to the overall test set — 60% of benign DBD variants are misclassified as pathogenic. This is a genuine, disclosed limitation, not an artifact: benign and pathogenic hotspot substitutions co-occur at adjacent DBD codons, making discrimination from primary sequence features alone intrinsically difficult in this region.

---

# Ablation Study

Each feature subset evaluated on the identical held-out test split as the primary model.

| Feature Subset | AUC-ROC | Sensitivity | Specificity |
|---|---|---|---|
| Full feature set (k-mers + biological) | 0.9511 | 0.8780 | 0.9151 |
| Sequence k-mers only | 0.8246 | 0.7317 | 0.8160 |
| Biological/position features only | 0.8753 | 0.7073 | 0.9009 |

Both feature groups contribute independently; neither alone approaches full-model performance.

---

# Comparative Benchmark

## CADD v1.7 (complete 294-variant test set)

Genomic coordinates (GRCh38, via Canonical SPDI) were submitted directly to the CADD GRCh38-v1.7 scoring server, with coordinate identity confirmed by index-verified matching against the original dataset prior to score retrieval.

| Tool | AUC-ROC | Sensitivity | Specificity | n |
|---|---|---|---|---|
| **This Model (RF)** | **0.9511** | **0.8780** | **0.9151** | **294** |
| CADD v1.7 (PHRED ≥ 25) | 0.9804 | 0.7195 | 0.9953 | 294 |

CADD's performance is markedly threshold-dependent:

| PHRED threshold | Sensitivity | Specificity |
|---|---|---|
| ≥ 20 | 0.9512 | 0.9623 |
| ≥ 25 | 0.7195 | 0.9953 |
| ≥ 30 | 0.3049 | 1.0000 |

The RF classifier, calibrated at a single fixed threshold of 0.5, exceeds CADD's sensitivity at CADD's conventional clinical operating point (PHRED ≥ 25), at the cost of modestly lower specificity, without requiring post hoc threshold selection.

## ClinPred, SIFT, and PolyPhen-2 (via dbNSFP v4, true missense subset)

CADD scores any nucleotide substitution, but SIFT, PolyPhen-2, and ClinPred are missense-only tools. The true missense subset was identified using ClinVar's native `Molecular consequence` field (case-insensitive substring match for `"missense"`, `na=False`), applied directly to this model's own saved test partition — **75 missense variants** (45 pathogenic, 30 benign). Scores were retrieved via the [myvariant.info](https://myvariant.info) API (`assembly=hg38`), which mirrors dbNSFP v4.

| Tool | AUC-ROC | Sensitivity | Specificity | n |
|---|---|---|---|---|
| **This Model (RF) — missense subset** | **0.8563** | **0.9778** | **0.5333** | **75** |
| ClinPred (dbNSFP) | 0.9637 | 0.9778 | 0.5333 | 75 |
| PolyPhen-2 (strict, D only) | 0.8899 | 0.9302 | 0.6000 | 73 |
| PolyPhen-2 (broad, D+P) | — | 0.9767 | 0.4333 | 73 |
| SIFT | 0.7915 | 1.0000 | 0.3333 | 73 |

\* SIFT and PolyPhen-2 categorical predictions use a conservative worst-case rule (most damaging prediction across all annotated TP53 transcript isoforms), which likely inflates the apparent false-positive rate relative to canonical-transcript-only scoring. n = 73 rather than 75 for these two tools because 2 variants lack dbNSFP transcript-level scores (both pathogenic).

**On this true missense subset, the RF classifier is not the strongest performer**: it ranks third of four on AUC (behind ClinPred and PolyPhen-2) and its specificity (0.5333) exceeds only SIFT's. This is disclosed here as a genuine limitation of missense-only discrimination, not minimized by leading with the headline full-test-set numbers above.

> **Note on an earlier reported figure:** a prior development version of this benchmark reported n = 72 for the missense subset. That figure was produced by independently re-deriving the train/test split from a differently sized dataframe (1,470 rows, prior to SPDI-based quality filtering) rather than reusing this model's actual saved test partition (1,469 rows). A stratified split re-run on a different-sized input is not guaranteed to reproduce identical test-set membership, even under an identical random seed. The n = 75 figures above use the model's actual test partition directly and are the correct values.

---

# Data Availability Note

CADD scores are not redistributed in this repository per CADD's terms of use. To reproduce the benchmark:

1. Run the pipeline notebook to generate the held-out test set from the trained model's exact 80/20 split (`random_state=42`).
2. Submit the test-set genomic coordinates to the [CADD scoring server](https://cadd.gs.washington.edu/score), selecting **GRCh38-v1.7**.
3. Merge the returned scores against test-set labels by index to reproduce the AUC, sensitivity, and specificity reported above.
4. For ClinPred/SIFT/PolyPhen-2, filter the *same* saved test partition to the true missense subset using ClinVar's `Molecular consequence` field, then query the myvariant.info API directly (`assembly=hg38`) — no file download required.

---

# Limitations

- Trained on a single ClinVar snapshot (April 2026); performance on subsequently reclassified variants is unknown.
- Variants of uncertain significance (46.3% of downloaded entries) were excluded and are not covered by reported performance.
- The "unknown domain" feature (intronic/splice-site variants lacking protein-change annotation) ranks among the top predictors by feature importance. Because intronic ClinVar submissions are predominantly benign, this is partly an annotation-availability confound rather than pure biological signal — the missense-only benchmark above isolates performance when this shortcut is unavailable, and specificity drops accordingly.
- Specificity within the DNA-binding domain (40.0%) and on missense-only variants (53.3%) is substantially weaker than overall test-set specificity (91.5%), reflecting the intrinsic difficulty of discriminating co-located pathogenic and benign substitutions from sequence context alone.
- Germline and somatic TP53 variants were not distinguished during training, despite mechanistically distinct pathogenic processes.
- Protein structural features (e.g., AlphaFold-derived solvent accessibility, contact frequencies) were not incorporated and would likely improve within-DBD sensitivity specifically.
- South Asian populations are substantially underrepresented in ClinVar, limiting direct applicability to underrepresented patient cohorts.
- This model is a research triage tool and is **not validated for clinical diagnostic use**.

---

# Reproducibility

All analyses were performed in Google Colab using Python 3.12.

**Dependencies (versions confirmed against the actual runtime that produced the reported results):**
- scikit-learn 1.6.1
- pandas 2.2.2
- numpy 2.0.2
- biopython 1.87
- matplotlib 3.10.0
- seaborn 0.13.2
- requests (for dbNSFP/myvariant.info queries)

Random seed fixed at 42 throughout all stochastic operations, including the train/test split. Note: scikit-learn does not guarantee bit-for-bit reproducibility of `RandomForestClassifier` output across different library versions even with an identical `random_state` — reproduction attempts should match the scikit-learn version above, not only the seed.

---

# Citation

If you use this work, please cite:

Zaib, B.; Jan, S.W.; Begum, K.; Jamal, M.; Rahman, S. Gene-Specific Random Forest Modeling of TP53 Variant Pathogenicity Using Domain-Aware Biological Features. *Int. J. Mol. Sci.* (in review).

---

# License

This repository is released under the MIT License. ClinVar data is publicly available under NCBI's data use policies. CADD and dbNSFP scores are subject to their respective terms of use and are not redistributed here.

---

# Contact

For questions regarding this repository, contact the corresponding authors listed in the associated manuscript.
