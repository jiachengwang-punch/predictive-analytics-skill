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

## Agent ensemble

Beyond the linear 7-stage pipeline, the skill ships a **4-agent ensemble** that adds independent, often adversarial perspectives so an analysis gets thought through from more angles. Each agent has a portable role-prompt in `references/agents/` (the single source of truth — paste into any LLM) and, for Claude Code, a thin plugin wrapper in `agents/` that points to it.

| Agent | When | What it does |
|-------|------|--------------|
| **methodology-architect** | Before modeling | Analysis blueprint: task type, method tier by sample/feature constraints, split & validation plan, leakage-risk list |
| **model-diagnostician** | After a good score | Adversarially hunts failure: residual structure, subgroup performance, calibration, leakage suspicion |
| **tool-scout** | When selection is uncertain | **Searches the web** for a better-fitting library/model — LightGBM is the default, not automatically best; flags clearly when offline |
| **professor-reviewer** | Before delivery | Professor-perspective review + grade; sends methodology flaws back with a remediation checklist (does not auto-rerun) |

A rigorous loop: architect (plan) → stages 1–7 → diagnostician (find flaws) → revise → professor-reviewer (final gate) → iterate; call tool-scout whenever model choice is in doubt. In Claude Code, install the repo as a plugin to use the agents natively; in any other LLM, paste the matching `references/agents/*.md` as a role prompt.

> **tool-scout in practice — when to reach for CatBoost.** LightGBM is the workhorse default, but on **categorical-heavy data** (many high-cardinality categorical columns, few numeric features), **CatBoost** is often the better fit: it handles categoricals natively (no manual one-hot) and uses ordered target encoding that guards against target leakage, frequently edging out LightGBM on such datasets at the cost of slower training. Reach for it for churn / recommendation / transaction-style tables dominated by categorical fields; stick with LightGBM when data is large and mostly numeric or training speed matters. When in doubt, dispatch tool-scout to verify the current best fit for your specific data.

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
├── .claude-plugin/
│   └── plugin.json       # Plugin manifest (use the repo as a Claude Code plugin)
├── SKILL.md              # Claude entry point (YAML header)
├── METHODOLOGY.md        # Plain-prompt entry point (other LLMs)
├── agents/               # Claude plugin agent wrappers (point to references/agents/)
│   ├── methodology-architect.md
│   ├── model-diagnostician.md
│   ├── tool-scout.md
│   └── professor-reviewer.md
├── references/           # Shared platform-neutral core
│   ├── agents/           # Portable agent role-prompts (single source of truth)
│   │   ├── methodology-architect.md
│   │   ├── model-diagnostician.md
│   │   ├── tool-scout.md
│   │   └── professor-reviewer.md
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

- **Screen for leakage, proportional to risk.** For every feature, ask: "Is this truly available at prediction time, or is it a downstream result of the target?" Leakage inflates scores but destroys real predictive value. The screening question is always worth asking; how hard you investigate scales with data provenance (high-risk: temporal prediction, target-derived features, multi-table joins, full-data encoding — low-risk: clean cross-sectional data with exogenous features).
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
