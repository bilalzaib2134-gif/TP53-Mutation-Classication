# TP53-Mutation-Classication
Machine learning-based classification of TP53 gene mutations using ClinVar dataset and Random Forest with biological feature integration.
# TP53 Mutation Classification Using Machine Learning

# TP53 Mutation Classification — Random Forest Classifier

> **AI-Based Classification of TP53 Gene Mutations Using Random Forest and ClinVar: A Gene-Specific Approach with Biological Feature Integration**
>
> Bilal Zaib · Department of Microbiology, Abdul Wali Khan University Mardan (AWKUM), Lower Dir, KP, Pakistan

---

## 🔬 Overview

TP53 is mutated in approximately **50% of all human cancers**, yet reliably separating pathogenic from benign variants within ClinVar remains an unresolved bottleneck in clinical oncology. This repository provides a **gene-specific, biologically informed Random Forest classifier** trained on 1,470 ClinVar-curated TP53 single nucleotide variants (SNVs) to predict variant pathogenicity.

Unlike genome-wide tools (SIFT, PolyPhen-2, CADD), this model is built exclusively for TP53, allowing it to exploit the strong spatial clustering of oncogenic mutations within the DNA-binding domain — a signal that is typically diluted in genome-wide models.

> ⚠️ **This model is intended for research use only and does not constitute a clinical diagnostic tool.**

---

## 📊 Performance Summary

| Metric | Value |
|---|---|
| AUC-ROC (test set) | **0.9327** |
| AUPRC (test set) | **0.8303** |
| Cross-validation AUC (5-fold) | **0.9106 ± 0.0184** |
| Specificity | 0.9481 |
| Sensitivity | 0.6829 |
| PPV | 0.8358 |
| NPV | 0.8855 |

Evaluated on a stratified held-out test set of **294 variants** (20% of total dataset).

---

## 📁 Repository Structure

```
TP53_Mutation_Classification/
│
├── data/
│   └── clinvar_tp53_snv_april2026.tsv   # ClinVar download (TP53 SNVs, April 2026)
│
├── notebooks/
│   └── TP53_RF_Classifier.ipynb         # Full pipeline: retrieval → features → training → evaluation
│
├── src/
│   ├── feature_extraction.py            # k-mer + biological feature engineering
│   ├── train_model.py                   # Random Forest training script
│   └── evaluate_model.py                # Metrics, ROC, PR curves, feature importance
│
├── figures/
│   ├── feature_importance.png
│   ├── roc_pr_curves.png
│   ├── confusion_matrix.png
│   ├── domain_distribution.png
│   ├── cv_auc_folds.png
│   ├── hotspot_vs_nonhotspot.png
│   └── probability_distribution.png
│
├── results/
│   └── performance_metrics.csv          # All evaluation metrics (Tables 2 & 3 from paper)
│
├── requirements.txt
└── README.md
```

---

## 🧬 Methods Summary

### Data Source
- **ClinVar** (April 2026): `TP53[gene] AND single nucleotide variant[variant type]`
- 2,739 total records downloaded; **1,470 retained** after excluding VUS/conflicting labels
- Labels: Pathogenic/Likely Pathogenic → `1`; Benign/Likely Benign → `0`
- Reference sequence: **NM_000546.6** (NCBI, retrieved via Biopython Entrez)

### Features (276 total)
- **256 sequence features** — trinucleotide and tetranucleotide k-mer frequencies from a ±50 bp window around each variant (scikit-learn `CountVectorizer`)
- **20 biological features** — codon position, relative genomic location, 7 functional domain memberships (one-hot), hotspot status (6 canonical codons: R175, G245, R248, R249, R273, R282), reference nucleotide, alternative nucleotide, nucleotide change identity

### Model
- `RandomForestClassifier` (scikit-learn v1.3)
- 200 estimators, `min_samples_split=3`, `class_weight='balanced'`
- 80/20 stratified train/test split (`random_state=42`)
- 5-fold stratified cross-validation on training partition

---

## ⚙️ How to Reproduce

### 1. Clone the repository
```bash
git clone https://github.com/YOUR_USERNAME/TP53_Mutation_Classification.git
cd TP53_Mutation_Classification
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Run the full pipeline
Open and run the notebook end-to-end:
```bash
jupyter notebook notebooks/TP53_RF_Classifier.ipynb
```
Or run on **Google Colab** (recommended — no local setup required):

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/TP53_Mutation_Classification/blob/main/notebooks/TP53_RF_Classifier.ipynb)

> All random seeds are fixed at `42` throughout. Results are fully reproducible.

---

## 📦 Requirements

```
python>=3.10
scikit-learn==1.3
pandas==2.0
numpy==1.24
biopython==1.81
matplotlib==3.7
seaborn==0.12
jupyter
```

Install with:
```bash
pip install -r requirements.txt
```

---

## 📈 Key Findings

- **Relative genomic position** and **codon position** were the two strongest predictors (Gini importance ~0.10 and ~0.04), consistent with the known spatial clustering of oncogenic TP53 mutations in the DNA-binding domain (codons 102–292).
- **DNA-binding domain membership** ranked 3rd–4th in feature importance.
- Top informative k-mers: tetranucleotides `CTGA`, `CTGT` and trinucleotide `CGA` — reflecting enrichment of specific sequence states in pathogenic regions.
- High specificity (94.8%) makes the model suited for **variant triage** tasks, where minimizing false positives is practically valuable.

---

## ⚠️ Limitations

1. Trained on a **single ClinVar snapshot** (April 2026) — performance on future submissions is unverified.
2. **1,269 VUS records (46.3%)** were excluded — performance on ambiguous variants is unknown.
3. Full benchmark against SIFT, PolyPhen-2, and CADD on the complete test set was **not performed** (partial PolyPhen-2 benchmark on 88 TAD1 variants only).
4. **Protein structural features** (AlphaFold-derived) were not included.
5. Germline and somatic variants were **not stratified**.
6. South Asian cancer cohorts are **underrepresented in ClinVar**, limiting direct applicability to Pakistani patient populations.

---

## 🗺️ Planned Extensions

- [ ] Full benchmark vs. SIFT, PolyPhen-2, CADD on 294-variant test set
- [ ] Ablation study: remove positional features to isolate k-mer contribution
- [ ] Prospective validation on variants reclassified in ClinVar after April 2026
- [ ] Integration of AlphaFold structural features (solvent accessibility, stability)

---

## 📄 Citation

If you use this code or data in your work, please cite:

```bibtex
@article{zaib2026tp53,
  title   = {AI-Based Classification of TP53 Gene Mutations Using Random Forest and ClinVar: 
             A Gene-Specific Approach with Biological Feature Integration},
  author  = {Zaib, Bilal},
  journal = {[Journal Name]},
  year    = {2026},
  note    = {Preprint / Under Review}
}
```

---

## 📬 Contact

**Bilal Zaib**  
Department of Microbiology  
Abdul Wali Khan University Mardan (AWKUM), Lower Dir, KP, Pakistan  
📧 bilalzaib.microbio@gmail.com

---

## 🔓 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

Data sourced from NCBI ClinVar is publicly available and governed by [NCBI's data usage policies](https://www.ncbi.nlm.nih.gov/home/about/policies/).
