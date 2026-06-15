# NLP Capstone: Leakage-Controlled Topic Classification on 20 Newsgroups

Supervised 7-class topic classification on a technical subset of the 20 Newsgroups dataset, comparing a tuned TF-IDF + Logistic Regression baseline with a fine-tuned DistilBERT transformer under leakage-controlled preprocessing and a thread-aware train/validation/test split.

The model is designed as an **assistive topic routing tool**: it ranks and assigns posts to topic categories to support human review, not as an autonomous classifier for production deployment.

**Dataset note:** This project uses the original `20news-19997.tar.gz` archive, not `sklearn.datasets.fetch_20newsgroups`, because raw headers and thread references are required for leakage and split analysis.

![Python](https://img.shields.io/badge/Python-3.10-blue)
![DistilBERT](https://img.shields.io/badge/Model-DistilBERT-blueviolet)
![TF--IDF](https://img.shields.io/badge/Baseline-TF--IDF%20%2B%20LR-lightblue)
![Macro F1](https://img.shields.io/badge/Metric-Macro%20F1-green)
![Thread Split](https://img.shields.io/badge/Eval-Thread--Aware%20Split-orange)
***

## Key Results

**Final test setup**
- Split strategy: **thread-aware train / validation / test**
- Primary metric: **Macro F1** (handles class imbalance)
- Input regime: `text_subject_body_clean` (leakage-controlled)

**Held-out test performance**

| Model | Macro F1 |
|-------|----------:|
| TF-IDF + Logistic Regression | 0.8465 |
| **DistilBERT (fine-tuned)** | **0.8956** |

![Model comparison](outputs/figures/distil_bert_vs_tf_idf_test_performance.png)

**Final recommended model:** DistilBERT fine-tuned on `text_subject_body_clean`

**Main limitation:** `comp.sys.ibm.pc.hardware` remains the weakest class due to topical overlap with Mac hardware, electronics, graphics hardware, and Windows configuration threads.

***

## Why This Project Matters

Technical forum posts are noisy, short, and often quote-heavy — standard classification pipelines fail silently when they rely on metadata artifacts (e.g., header fields, newsgroup source markers) instead of actual content signal.

This project therefore frames the task as a **leakage-controlled** classification problem:
- **Preprocessing audit** removes source-specific artifacts and header leakage before any modelling
- **Thread-aware split** ensures posts from the same thread never appear in both train and test
- **Macro F1** is used as the primary metric to penalise poor performance on minority classes, not just aggregate accuracy

***

## Final Model

The final model is **DistilBERT**, selected after:
- Baseline modelling with TF-IDF + Logistic Regression across multiple input regimes
- Leakage and artifact audit (header fields, newsgroup markers, signature blocks)
- Input regime ablation: raw text vs. subject-only vs. cleaned combined body+subject
- Fine-tuning DistilBERT on the leakage-controlled `text_subject_body_clean` regime

DistilBERT outperformed the linear baseline across all classes, particularly on boundary cases where topical overlap makes bag-of-words representations insufficient.

***

## Main Findings

- **Transformer fine-tuning outperformed the linear baseline** by ~5 macro F1 points on the held-out test set.
- **Leakage audit materially affected results** — naive preprocessing using raw text with header fields inflated baseline scores artificially.
- **`comp.sys.ibm.pc.hardware` is the persistent weak class**: boundary confusion with adjacent hardware and OS topics is a structural data problem, not a modelling failure.
- **Short, low-confidence, and quote-heavy posts** form a distinct error pattern and warrant human review rather than automated routing.

![Class cosine similarity](outputs/figures/class_cosine_similarity.png)

**Recommended deployment safeguards:**
- minimum confidence threshold before automated routing
- flagging of short posts, high-quote-ratio posts, and hardware-boundary predictions for human review
- periodic monitoring for distribution drift across topic categories

***

## Method Overview

1. **EDA and preprocessing**
   - dataset inspection and class distribution analysis
   - leakage and artifact audit (headers, footers, source markers)
   - empirical data dictionary construction
   - cleaned text regime creation (`text_subject_body_clean`)

2. **Baseline modelling**
   - TF-IDF + Logistic Regression
   - input regime ablation across raw and cleaned variants
   - error analysis per class

3. **Transformer fine-tuning**
   - DistilBERT on `text_subject_body_clean`
   - thread-aware train/validation/test split
   - final evaluation on frozen held-out test set

4. **Evaluation and explainability**
   - per-class F1 analysis
   - error analysis for misclassified posts
   - LIME explainability outputs
   - risk assessment and deployment recommendation

***

## Repository Structure

```text
├── notebooks/
│   ├── 01_20newsgroup_EDA.ipynb         # Dataset inspection, preprocessing, leakage audit, text regimes
│   └── 02_20newsgroup_Modeling.ipynb    # Split strategy, baseline, DistilBERT, evaluation, LIME, risk
├── outputs/
│   ├── config/
│   │   ├── final_run_config.json
│   │   ├── environment_info.json
│   │   ├── requirements.txt
│   │   └── output_manifest.json
│   ├── tables/                          # Final test metrics, per-class results, split assignments, explainability
│   └── figures/                         # Key plots used in the report
└── README.md
```

***

## Reproducibility Notes

The project saves environment information, final run configuration, split assignments, reported metrics, predictions, figures, and explainability outputs. The final test set is used only after preprocessing decisions, model selection, and hyperparameter choices are frozen.

The raw 20 Newsgroups data is not included. Processed files and split assignments are provided where needed to trace reported results.

> To reproduce: run notebooks in order (`01` → `02`). All preprocessing and split logic is documented inline. No external data download required beyond the standard `sklearn.datasets.fetch_20newsgroups` loader.
