# Predictive Analytics Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Release](https://img.shields.io/github/v/release/jiachengwang-punch/predictive-analytics-skill)](https://github.com/jiachengwang-punch/predictive-analytics-skill/releases)
![Claude Skill](https://img.shields.io/badge/Claude-Skill-8A2BE2)
![Multi-model](https://img.shields.io/badge/LLM-multi--model-blue)
![Language-adaptive](https://img.shields.io/badge/output-language--adaptive-green)

A complete, reusable, **multi-model** and **language-adaptive** methodology for end-to-end machine learning analysis of structured (tabular) data — from raw data to a validated, interpretable predictive model and an honest, decision-oriented report.

The methodology is **domain-agnostic**: it works for customer churn, sales/demand forecasting, risk/default prediction, site selection, sensor/monitoring data, and any other "I have a dataset and want to predict or understand Y" task. A city-noise dataset is used as the running example throughout the guides.

## Why this exists

Most data-analysis tutorials stop at "run a model and report the score." This skill encodes the *analysis spirit* that separates rigorous work from score-chasing: detect data leakage before anything else, always compare against a baseline, cross-validate, diagnose where the model fails, and explain the result. It packages that discipline into a form any LLM can follow.

## Dual-format architecture

Two entry points, one shared core — so the same methodology runs on any model:

```
                 Two entry points — same methodology
        ┌────────────────────────┬────────────────────────┐
        │  SKILL.md              │  METHODOLOGY.md          │
        │  (YAML header)         │  (plain prompt)          │
        │  → for Claude          │  → for any other LLM     │
        └───────────┬────────────┴───────────┬─────────────┘
                    │         shares          │
                    ▼                         ▼
              ┌──────────────────────────────────────┐
              │  references/ — platform-neutral core  │
              │  7 deep-dive guides (one per stage)   │
              └───────────────┬──────────────────────┘
                    │                         │
                    ▼                         ▼
            ┌───────────────┐       ┌──────────────────────────┐
            │  Claude        │       │  ChatGPT · Gemini         │
            │  install as    │       │  DeepSeek · Qwen · Kimi   │
            │  Skill or paste │       │  any LLM via API          │
            └───────────────┘       └──────────────────────────┘
```

- **`SKILL.md`** — Claude entry point. Has a YAML header so it installs directly as a Claude Skill.
- **`METHODOLOGY.md`** — plain-prompt entry point. Paste it into any other LLM (ChatGPT, Gemini, DeepSeek, Qwen, Kimi, or any model via API) as a system prompt or conversation opener.
- **`references/`** — the shared, platform-neutral core: seven deep-dive guides, one per stage. Both entry points point to these. The methodology and its rigor are identical across models; only the loading mechanism differs.

## Language support

Both entry points carry a highest-priority instruction: **detect the language of the user's request and produce the entire analysis in that language** — section titles, explanations, result interpretations, chart labels, and the final report. Ask in English, get English; ask in Chinese, get Chinese (including matplotlib chart labels).

## The 7-stage workflow

| Stage | Guide | What it covers |
|-------|-------|----------------|
| 1. Data import & cleaning | `references/01_data_import_cleaning.md` | Load data, understand fields, quality checks, **data-leakage detection**, time/type handling |
| 2. Exploratory data analysis | `references/02_eda.md` | Distributions, group comparisons, correlations, time/space/category patterns, chart-font setup |
| 3. Feature engineering | `references/03_feature_engineering.md` | Remove leakage/ID features, derive features, encode categoricals, feature–target analysis |
| 4. Modeling | `references/04_modeling.md` | Baseline → ensemble progression, classification & regression, class-imbalance handling |
| 5. Evaluation & diagnosis | `references/05_evaluation_diagnosis.md` | Right metrics, K-fold CV, grid search, **residual diagnosis**, data-driven thresholds |
| 6. Clustering | `references/06_clustering.md` | Profile aggregation, KMeans, **PCA for honest high-dim visualization** |
| 7. Interpretability & output | `references/07_interpretability_output.md` | Feature importance, SHAP, honest model math, report composition |

## Installation & usage

Repository: <https://github.com/jiachengwang-punch/predictive-analytics-skill>

```bash
git clone https://github.com/jiachengwang-punch/predictive-analytics-skill.git
```

### For Claude

**Option A — install as a Skill (recommended):** package the folder into a `.skill` file and install it via Claude's skill settings. Once installed, it triggers automatically when you ask to analyze a dataset or build a model.

**Option B — paste in chat:** open `SKILL.md` and paste its contents into the conversation, then describe your task and attach your data.

### For other LLMs (ChatGPT, Gemini, DeepSeek, Qwen, Kimi, etc.)

Open `METHODOLOGY.md`, copy its entire contents, and paste it into the model as a system prompt or at the start of your conversation. Then describe your data-analysis task and provide your data (or its description). If your environment supports file uploads, you can also attach the relevant `references/` guide for the stage you're working on.

## Repository structure

```
predictive-analytics-skill/
├── SKILL.md              # Claude entry point (YAML header)
├── METHODOLOGY.md        # Plain-prompt entry point (other LLMs)
├── references/           # Shared platform-neutral core (7 guides)
│   ├── 01_data_import_cleaning.md
│   ├── 02_eda.md
│   ├── 03_feature_engineering.md
│   ├── 04_modeling.md
│   ├── 05_evaluation_diagnosis.md
│   ├── 06_clustering.md
│   └── 07_interpretability_output.md
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── CHANGELOG.md
└── .gitignore
```

## Core principles (the "analysis spirit")

These apply at every stage and are the heart of the methodology:

- **Detect leakage first.** For every feature, ask: "Is this truly available at prediction time, or is it a downstream result of the target?" Leakage inflates scores but destroys real predictive value.
- **Honesty over a pretty number.** Report negative results, expose where the model fails, state limitations.
- **Always compare against a baseline.** A complex model must beat a simple one to justify itself.
- **Match method to constraint.** Sample size, data type, and interpretability needs drive method choice — not fashion.
- **Let the data tell you thresholds.** Use data-driven splits over guessed cutoffs — but remember tree models auto-find splits, so manual binning helps linear models, not trees.
- **Cross-validate and interpret.** Don't trust a single split; don't ship a black box without explanation.

## Tech stack (for the code patterns in the guides)

Python with `pandas`, `scikit-learn`, `lightgbm`, `shap`, `matplotlib`, `seaborn`, `scipy`. The guides contain copy-adaptable code patterns; adapt them to your dataset and language.

## License

Released under the MIT License. See [LICENSE](LICENSE).

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).
