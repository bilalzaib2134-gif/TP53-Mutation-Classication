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

The classifier was compared against **CADD v1.7** using only variants confirmed to belong to the independent held-out test set.

| Tool | AUC |
|------|------|
| Random Forest | **0.9183** |
| CADD v1.7 | **0.7560** |

Benchmark performed on **31 matched test-set variants**.

SIFT and PolyPhen-2 were not quantitatively compared because too few matched test variants were available for reliable AUC estimation.

---

# Repository Structure

```
TP53-Mutation-Classification/

├── data/
├── notebooks/
├── src/
├── figures/
├── results/
├── requirements.txt
└── README.md
```

---

# Installation

```bash
git clone https://github.com/bilalzaib2134-gif/TP53-Mutation-Classification.git

cd TP53-Mutation-Classification

pip install -r requirements.txt
```

---

# Running

Open

```
notebooks/TP53_RF_Classifier.ipynb
```

Run every notebook cell sequentially.

All analyses are fully reproducible.

Random seed:

```
42
```

---

# Key Findings

- Relative genomic position was the strongest predictor.
- Codon position ranked second.
- DNA-binding domain membership was highly informative.
- Sequence k-mer features contributed additional discriminatory information.
- The TP53-specific classifier outperformed the genome-wide CADD predictor on the independent benchmark.

---

# Limitations

- Single ClinVar snapshot (April 2026)
- Variants of uncertain significance were excluded
- CADD benchmark restricted to 31 matched held-out variants
- AlphaFold structural features not included
- Germline and somatic variants not separated
- Limited representation of South Asian populations

---

# Future Work

- Ablation study removing positional features
- Larger benchmark against external predictors
- Validation using future ClinVar releases
- Integration of AlphaFold-derived structural features

---

# Citation

```bibtex
@article{zaib2026tp53,
  author = {Bilal Zaib and Syed Waleed Jan and Khaist Begum and Muhsin Jamal and Sajid ur Rahman},
  title = {AI-Based Classification of TP53 Gene Mutations Using Random Forest and ClinVar: A Gene-Specific Approach with Biological Feature Integration},
  journal = {International Journal of Molecular Sciences},
  year = {2026},
  note = {Submitted / Under Review}
}
```

---

# Contact

Bilal Zaib

Department of Microbiology

Abdul Wali Khan University Mardan

Pakistan

Email:

bilalzaib.microbio@gmail.com

---

# License

MIT License

The ClinVar dataset is publicly available from NCBI and remains subject to NCBI data usage policies.
