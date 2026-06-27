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

- Total variants: **2,739**
- Included variants: **1,470** (410 pathogenic, 1,060 benign)
- Excluded VUS/conflicting variants: **1,269** (46.3%)

Binary labels:

- Pathogenic / Likely Pathogenic → 1
- Benign / Likely Benign → 0

Reference transcript: `NM_000546.6`

---

# Feature Engineering

Total features: **276**

**Sequence features (256)**
- Trinucleotide and tetranucleotide k-mer frequencies
- 101 bp sequence window (±50 bp), centered on the alternate allele

**Biological features (20)**
- Codon position
- Relative genomic position
- Functional domain membership (7 domains: TAD1, TAD2, PRR, DBD, Linker, TET, REG)
- Hotspot status (6 canonical codons: R175, G245, R248, R249, R273, R282)
- Reference / alternate nucleotide identity
- Nucleotide substitution type

---

# Machine Learning Model

**Classifier:** `RandomForestClassifier` (scikit-learn)

**Parameters:**
- 200 trees
- `class_weight="balanced"`
- `min_samples_split=3`
- `random_state=42`

**Evaluation:**
- 80/20 stratified train/test split (1,176 / 294 variants)
- Five-fold stratified cross-validation
- Independent held-out test set, evaluated exactly once

---

# Performance

| Metric | Value |
|---|---|
| Test AUC-ROC | **0.9183** |
| Test AUPRC | **0.8359** |
| Five-fold CV AUC | **0.9002 ± 0.0232** |
| Sensitivity | **0.6951** |
| Specificity | **0.9481** |
| PPV | **0.8382** (83.8%) |
| NPV | **0.8894** (88.9%) |

Held-out test set: **294 TP53 variants** (82 pathogenic, 212 benign)

---

# Comparative Benchmark

## CADD v1.7

Compared against the complete 294-variant held-out test set. Genomic coordinates (GRCh38, via Canonical SPDI) were submitted directly to the CADD GRCh38-v1.7 scoring server, with coordinate identity confirmed by index-verified matching against the original dataset prior to score retrieval — no intermediate annotation file was used.

| Tool | AUC-ROC | Sensitivity | Specificity | n |
|---|---|---|---|---|
| **This Model (RF)** | **0.9183** | **0.6951** | **0.9481** | **294** |
| CADD v1.7 (PHRED ≥ 25) | 0.9814 | 0.7195 | 0.9953 | 294 |

CADD's performance is markedly threshold-dependent:

| PHRED threshold | Sensitivity | Specificity |
|---|---|---|
| ≥ 20 | 0.9512 | 0.9623 |
| ≥ 25 | 0.7195 | 0.9953 |
| ≥ 30 | 0.3049 | 1.0000 |

The RF classifier, calibrated at a single fixed threshold of 0.5, achieves performance comparable to CADD's PHRED 25 operating point without requiring post hoc threshold selection.

## ClinPred, SIFT, and PolyPhen-2 (via dbNSFP v4)

CADD scores any nucleotide substitution, but SIFT, PolyPhen-2, and ClinPred are missense-only tools. Of the 294 test variants, **72 are missense** (45 pathogenic, 27 benign) — the maximum possible coverage for these tools. Scores were retrieved via the [myvariant.info](https://myvariant.info) API, which mirrors dbNSFP v4.

| Tool | AUC-ROC | Sensitivity | Specificity | n |
|---|---|---|---|---|
| ClinPred (dbNSFP) | 0.9802 | 0.9778 | 0.7778 | 72 |
| SIFT* | — | 1.0000 | 0.4074 | 70 |
| PolyPhen-2 (D only)* | — | 0.9302 | 0.6667 | 70 |

\* SIFT and PolyPhen-2 return only categorical predictions (no continuous score), and TP53 has multiple annotated transcript isoforms with non-aligned per-transcript scores in dbNSFP. A conservative worst-case rule (most damaging prediction across all annotated transcripts) was applied, which likely inflates the apparent false-positive rate relative to canonical-transcript-only scoring. **These two results are exploratory and should not be read as a clean head-to-head comparison.**

---

# Data Availability Note

CADD scores are not redistributed in this repository per CADD's terms of use. To reproduce the benchmark:

1. Run `notebooks/extract_test_vcf.ipynb` to generate the held-out test-set VCF from the trained model's exact 80/20 split (`random_state=42`).
2. Submit the VCF to the [CADD scoring server](https://cadd.gs.washington.edu/score), selecting **GRCh38-v1.7**.
3. Download the scored TSV and run `notebooks/cadd_benchmark.ipynb` to merge scores against test-set labels and reproduce the AUC, sensitivity, and specificity reported above.
4. For ClinPred/SIFT/PolyPhen-2, run `notebooks/dbnsfp_benchmark.ipynb`, which queries the missense subset of the test set against the myvariant.info API directly — no file download required.

---

# Limitations

- Trained on a single ClinVar snapshot (April 2026); performance on subsequently reclassified variants is unknown, particularly relevant given documented improvements in ClinVar classification accuracy over time.
- Variants of uncertain significance (46.3% of downloaded entries) were excluded and are not covered by reported performance.
- The "unknown domain" feature (variants lacking protein-change annotation) ranked unexpectedly high in feature importance, an artifact possibly linked to broader, documented inconsistencies in variant annotation completeness.
- Germline and somatic TP53 variants were not distinguished during training, despite mechanistically distinct pathogenic processes.
- Protein structural features (e.g., AlphaFold-derived solvent accessibility, contact frequencies) were not incorporated and would likely improve within-DBD sensitivity specifically.
- South Asian populations are substantially underrepresented in ClinVar, limiting direct applicability to underrepresented patient cohorts.
- This model is a research triage tool and is **not validated for clinical diagnostic use**, particularly for germline interpretation in contexts such as Li-Fraumeni syndrome.

---

# Reproducibility

All analyses were performed in Google Colab using Python 3.12.

**Dependencies:**
- scikit-learn 1.3.2
- pandas 2.1.4
- numpy 1.26.4
- biopython 1.81
- matplotlib 3.7.2
- seaborn 0.12.2
- requests (for dbNSFP/myvariant.info queries)

Random seed fixed at 42 throughout all stochastic operations, including the train/test split.

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
