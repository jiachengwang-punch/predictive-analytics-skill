---
name: predictive-analytics
description: >
  A complete, reusable methodology for end-to-end machine learning analysis of structured (tabular) data — data import and cleaning, exploratory analysis, feature engineering, classification/regression modeling, model diagnosis, clustering, interpretability, and final reporting. Use this skill WHENEVER the user wants to analyze a tabular dataset (CSV/Excel), build a predictive model (classification or regression), do EDA, engineer features, evaluate or diagnose a model, cluster/segment records, or produce a data-analysis report or thesis — even if they ask for just one piece (e.g. "build a churn model", "do EDA on this CSV", "what features should I use", "cluster my stores"). Trigger for any "I have a dataset and want to predict/understand Y" task: customer churn, sales/demand forecasting, risk/default prediction, site selection, sensor/monitoring data, and similar. Domain-agnostic; a city-noise dataset is the running example.
---

# 预测分析方法论 / Predictive Analytics Methodology

> A platform-neutral, language-adaptive methodology for taking any structured (tabular) dataset through a rigorous, complete machine-learning analysis. Suitable for coursework, theses, and real business analysis.

## ⚠️ HIGHEST-PRIORITY INSTRUCTION: Output language

**Detect the language of the user's request and produce the ENTIRE analysis in that language** — section titles, explanations, result interpretations, chart text/labels, and the final report. If the user writes in Chinese, output everything in Chinese (including matplotlib chart labels). If in English, output in English. If in another language, match it. Code comments should also follow the user's language where practical. Do not switch languages mid-analysis. This instruction overrides any default.

## What this methodology delivers

A complete analysis pipeline that turns raw tabular data into an interpretable, validated predictive model plus an honest, decision-oriented report. It covers both **classification** and **regression**, plus **unsupervised clustering** as a complementary view. It bakes in research-grade rigor: data-leakage detection, baseline comparison, cross-validation, residual diagnosis, and data-driven (not guessed) thresholds.

## The 7-stage workflow

Work through these stages in order. Each has a dedicated deep-dive guide in `references/`. Read the relevant guide before executing that stage — the guides contain the concrete techniques, code patterns, and the *reasoning* behind each choice.

1. **Data import & cleaning** → `references/01_data_import_cleaning.md`
   Load data; understand every field; check missing/duplicate/anomalies; **screen for data leakage proportional to risk** (always ask the question; investigate deeply when provenance is risky); fix time/type issues.

2. **Exploratory data analysis (EDA)** → `references/02_eda.md`
   Understand what drives the target before modeling: distributions, group comparisons, correlations, time/space/category patterns. Produces the charts that tell the data's story.

3. **Feature engineering** → `references/03_feature_engineering.md`
   The soul of the analysis. Remove leakage/ID features, derive domain-meaningful features, encode categoricals, analyze feature-target relationships. For low-feature data, this is where value is created.

4. **Modeling (classification & regression)** → `references/04_modeling.md`
   Baseline → ensemble progression (logistic/linear → random forest → gradient boosting). Handle class imbalance. Choose the right task type.

5. **Evaluation, validation & diagnosis** → `references/05_evaluation_diagnosis.md`
   Use the right metrics (AUC/PR for imbalanced classification; R²/RMSE/MAE for regression — never just accuracy). K-fold cross-validation for stability. Grid search for tuning. **Residual diagnosis to actively expose model flaws** — don't stop at a good score.

6. **Clustering (complementary unsupervised view)** → `references/06_clustering.md`
   Segment records (e.g. customers, stores, sensors) by their profiles. RFM/feature aggregation + KMeans. **Use PCA for honest high-dimensional cluster visualization** (a 2-D scatter of a 5-D clustering misleads).

7. **Interpretability, math expression & output** → `references/07_interpretability_output.md`
   Feature importance + SHAP to answer "which factors matter, in which direction". Honest model math (linear models have equations; tree ensembles do NOT — describe their additive structure instead). Compose the final report and deliverables.

## Cross-cutting principles (the "analysis spirit")

These apply at every stage and are what separate a rigorous analysis from "just running a model":

- **Screen for leakage, proportional to risk.** Always ask of every feature: "Is this truly available at prediction time, or is it a result/consequence of the target?" A feature that is a downstream effect of the label inflates scores but destroys real predictive value. The *screening question* is near-zero-cost and always worth asking — but how hard you investigate should scale with data provenance: temporal prediction, target-derived features, multi-table joins, or full-data encoding/scaling are high-risk and warrant deep checks; clean cross-sectional data with clearly exogenous features (a curated benchmark, independent sensor readings) is low-risk — confirm and move on rather than manufacturing a leakage hunt. This is "match method to constraint" applied to QC.
- **Honest reporting over a pretty number.** Report negative results, expose where the model fails (residual diagnosis), state limitations. A model with R²=0.97 that systematically under-predicts the extreme cases can be worse in practice than its score suggests.
- **Always compare against a baseline.** A complex model must beat a simple one (logistic/linear regression, or "predict the mean") to justify itself.
- **Match method to constraint.** Sample size, data type, and interpretability needs drive method choice — not fashion. Large tabular data → gradient-boosted trees usually win; tiny samples → simple regularized linear models; non-tabular (images/text) → deep learning. Don't apply small-sample tactics (heavy regularization, LOOCV) to big data, or vice versa.
- **Let the data tell you thresholds.** Replace guessed cutoffs (e.g. "traffic > 60 = congested") with data-driven splits (decision-tree split points). But note: tree models auto-find splits, so manual binning helps linear models, not trees — the value of feature engineering depends on the model type.
- **Cross-validate and interpret.** Don't trust a single train/test split; don't ship a black box without SHAP/importance explanation.

## How to use the references

Each reference is self-contained and platform-neutral. Read the guide for the stage you're working on; it explains *why* each step matters (theory of mind), gives concrete code patterns (Python: pandas, scikit-learn, lightgbm, shap, matplotlib), and shows how to interpret outputs. Adapt all examples to the user's actual dataset and language.

## Agent ensemble (for more comprehensive analysis)

Four specialized agents add independent, often adversarial perspectives so the analysis is thought through from more angles than a single linear pass. Each has a portable role-prompt in `references/agents/` (the single source of truth, usable in any LLM) and, for Claude Code, a thin plugin wrapper in `agents/` that points to it. Dispatch them via the Task tool at the right moment — they are optional reinforcements, not required steps.

| Agent | When to dispatch | What it does |
|-------|------------------|--------------|
| **methodology-architect** | Before modeling (before stage 4) | Produces an analysis blueprint: task type, method tier by sample-size/feature constraint, split & validation strategy, leakage-risk list. |
| **model-diagnostician** | After a model scores well (after stage 5) | Adversarially hunts for failure: residual structure, subgroup performance, calibration, error clusters, suspiciously-high-score leakage suspicion. |
| **tool-scout** | When tool/model choice is uncertain | **Searches the web** to verify whether a better-fitting library/model exists for this scenario — LightGBM is the default, not automatically the best. Returns scenario-based recommendations + sources; degrades to a knowledge-based (clearly-flagged) recommendation when offline. |
| **professor-reviewer** | Before final delivery | Reviews the whole deliverable as a university data-analysis professor: grade + commentary; methodology flaws (leakage, no baseline, wrong metric) get sent back with a concrete remediation checklist. Does **not** auto-rerun — the main flow decides whether to iterate, then can re-review until it passes. |

A natural rigorous loop: **methodology-architect** (plan) → run stages 1–7 → **model-diagnostician** (find flaws) → revise if needed → **professor-reviewer** (final gate) → iterate until deliverable-quality. Call **tool-scout** whenever selection is in doubt. For other LLMs without subagents, paste the relevant `references/agents/*.md` as a role prompt to get the same perspective.

## Output & deliverables

When the user wants packaged deliverables, produce up to four, per `references/07`:
1. A complete notebook / analysis (all stages, with results and charts).
2. A written report or thesis draft (structured prose with embedded charts and tables).
3. Modular code files (one per stage).
4. A charts folder (all figures saved as images).

Always confirm scope and the prediction target with the user before a full build, and state assumptions inline rather than over-asking.

## Multi-model note

This skill (`SKILL.md`) is the Claude entry point. The same methodology is also packaged as `METHODOLOGY.md` — a plain-prompt version that can be pasted into other LLMs (ChatGPT, Gemini, DeepSeek, Qwen, Kimi, or any model via API). Both entry points share the same `references/` core. The methodology and its rigor are identical across models; only the loading mechanism differs.
