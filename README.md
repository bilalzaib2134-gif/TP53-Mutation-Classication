# TP53 Mutation Classification Using Machine Learning

> **AI-Based Classification of TP53 Gene Mutations Using Random Forest and ClinVar: A Gene-Specific Approach with Biological Feature Integration**

Machine learning-based classification of TP53 single nucleotide variants using a biologically informed Random Forest classifier trained on expert-curated ClinVar data.

> **Research use only. Not intended for clinical diagnosis.**

---

# Overview

TP53 is altered in approximately half of all human cancers, yet distinguishing pathogenic from benign variants remains a major challenge in clinical genomics.

This repository contains a gene-specific Random Forest classifier trained exclusively on TP53 variants. Unlike genome-wide pathogenicity predictors, this model integrates local sequence context together with biologically meaningful features including codon position, functional domain membership, hotspot status, genomic location, and nucleotide substitution information.

The complete workflow is fully reproducible using open-source software and Google Colab.

---

# Dataset

Source:

- NCBI ClinVar (April 2026)
- Query:
  TP53[gene] AND single nucleotide variant[variant type]

Downloaded variants:

- Total variants: **2,739**
- Included variants: **1,470**
- Excluded VUS/conflicting variants: **1,269**

Binary labels:

- Pathogenic / Likely Pathogenic → 1
- Benign / Likely Benign → 0

Reference transcript:

NM_000546.6

---

# Feature Engineering

Total features: **276**

Sequence features (256)

- Trinucleotide k-mer frequencies
- Tetranucleotide k-mer frequencies
- 101 bp sequence window (±50 bp)

Biological features (20)

- Codon position
- Relative genomic position
- Functional domain membership
- Hotspot status
- Reference nucleotide
- Alternate nucleotide
- Nucleotide substitution type

---

# Machine Learning Model

Classifier:

RandomForestClassifier

Parameters

- 200 trees
- class_weight="balanced"
- min_samples_split=3
- random_state=42

Evaluation

- 80/20 stratified train/test split
- Five-fold stratified cross-validation
- Independent held-out test set

---

# Performance

| Metric | Value |
|---------|-------|
| Test AUC-ROC | **0.9183** |
| Test AUPRC | **0.8359** |
| Five-fold CV AUC | **0.9002 ± 0.0232** |
| Sensitivity | **0.6951** |
| Specificity | **0.9481** |
| PPV | **0.8382** |
| NPV | **0.8894** |

Held-out test set:

**294 TP53 variants**

---

# Comparative Benchmark

The classifier was compared against **CADD v1.7** on the complete 294-variant held-out test set. Genomic coordinates were submitted directly to the CADD GRCh38-v1.7 web server, with coordinate identity confirmed by index-verified matching against the original dataset prior to score retrieval.

| Tool | AUC-ROC | Sensitivity | Specificity | n |
|------|---------|-------------|-------------|---|
| **This Model (RF)** | **0.9183** | **0.6951** | **0.9481** | **294** |
| CADD v1.7 (PHRED ≥ 25) | 0.9814 | 0.7195 | 0.9953 | 294 |

CADD's performance is highly threshold-dependent:

| PHRED threshold | Sensitivity | Specificity |
|------------------|-------------|--------------|
| ≥ 20 | 0.9512 | 0.9623 |
| ≥ 25 | 0.7195 | 0.9953 |
| ≥ 30 | 0.3049 | 1.0000 |

The RF classifier, calibrated at a single fixed threshold of 0.5, achieves performance comparable to CADD at its PHRED 25 operating point without requiring threshold selection.

SIFT and PolyPhen-2 annotation coverage within the held-out test set was insufficient for reliable AUC estimation (fewer than 10 matched variants with severe class imbalance) and are not reported quantitatively.

---

# Data Availability Note

CADD scores are not redistributed in this repository per CADD's terms of use. To reproduce the benchmark:

1. Run `notebooks/extract_test_vcf.ipynb` to generate the held-out test-set VCF from the trained model's exact 80/20 split (`random_state=42`).
2. Submit the VCF to the [CADD scoring server](https://cadd.gs.washington.edu/score), selecting GRCh38-v1.7.
3. Download the scored TSV and run `notebooks/cadd_benchmark.ipynb` to merge scores against test-set labels and reproduce the AUC, sensitivity, and specificity reported above.

---

# Limitations

- Trained on a single ClinVar snapshot (April 2026); performance on subsequently reclassified variants is unknown.
- Variants of uncertain significance (46.3% of downloaded entries) were excluded and are not covered by reported performance.
- Germline and somatic variants were not distinguished during training.
- South Asian populations are substantially underrepresented in ClinVar, limiting direct applicability to underrepresented patient cohorts.
- This model is a research triage tool and is not validated for clinical diagnostic use.

---

# Reproducibility

All analyses were performed in Google Colab using Python 3.12.

Dependencies:

- scikit-learn 1.3.2
- pandas 2.1.4
- numpy 1.26.4
- biopython 1.81
- matplotlib 3.7.2
- seaborn 0.12.2

Random seed fixed at 42 throughout all stochastic operations.

---

# Citation

If you use this work, please cite:

Zaib, B.; Jan, S.W.; Begum, K.; Jamal, M.; Rahman, S. AI-Based Classification of TP53 Gene Mutations Using Random Forest and ClinVar: A Gene-Specific Approach with Biological Feature Integration. *Int. J. Mol. Sci.* (in review).

---

# License

This repository is released under the MIT License. ClinVar data is publicly available under NCBI's data use policies.

---

# Contact

For questions regarding this repository, contact the corresponding authors listed in the associated manuscript.
