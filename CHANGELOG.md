# Changelog

All notable changes to this skill are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/), and this project adheres to
semantic versioning.

## [2.0.2] - 2026-05-30

### Changed
- README: added CatBoost selection guidance to the agent-ensemble section (tool-scout's practical recommendation from the Telco Churn validation run) — prefer CatBoost on categorical-heavy data, keep LightGBM for large mostly-numeric data or when training speed matters.

## [2.0.1] - 2026-05-30

### Changed
- `model-diagnostician`: reordered the leakage-recheck signals by reliability after an end-to-end validation run on the Telco Churn dataset — (1) anomalous score jump, (2) feature-vs-label consistency check (>95% ⇒ leakage), (3) importance via **gain/SHAP, not split-count**. Documented that a binary leakage feature can be buried under continuous features in default split-count importance, so split-count is unreliable for leakage detection. Updated both the portable role-prompt and the plugin wrapper.

## [2.0.0] - 2026-05-30

### Added
- 4-agent ensemble for more comprehensive, rigor-checked analysis:
  - `methodology-architect` — pre-modeling analysis blueprint (task type, method tier, validation plan, leakage-risk list).
  - `model-diagnostician` — adversarial post-evaluation failure hunting (residuals, subgroup performance, calibration, leakage suspicion).
  - `tool-scout` — web-search-backed library/model recommendation (LightGBM is the default, not automatically the best); degrades to a clearly-flagged offline recommendation.
  - `professor-reviewer` — professor-perspective final-delivery review with grade + remediation checklist; does not auto-rerun.
- Portable agent role-prompts in `references/agents/` (single source of truth, usable in any LLM) and Claude plugin wrappers in `agents/` that point to them.
- `.claude-plugin/plugin.json` so the repository can be used as a Claude Code plugin (agents available natively).
- Agent-ensemble sections in `SKILL.md`, `METHODOLOGY.md`, and `README.md`.
- `methodology-architect`: explicit deep-learning guidance — DL is a must-be-justified candidate on tabular data (GBDT usually wins), with signals for when it's worth considering and routing for non-tabular data.
- `model-diagnostician`: explicit overfitting check (train-vs-test gap, distinguished from leakage) and a note that diagnostics are model-family-agnostic (apply to DL too).

### Changed
- Reframed data-leakage detection from an absolute, always-mandatory step to **risk-proportional screening**: the screening question is always asked (near-zero cost), but investigation depth scales with data provenance (high-risk: temporal prediction, target-derived features, multi-table joins, full-data encoding/scaling — low-risk: clean cross-sectional data with exogenous features). Updated `references/01`, `references/03`, `SKILL.md`, `METHODOLOGY.md`, `README.md`, and the `methodology-architect` / `professor-reviewer` agents accordingly, aligning leakage QC with the existing "match method to constraint" principle.

## [1.1.0] - 2026-05-26

### Changed
- Renamed the skill from `structured-data-analysis` to `predictive-analytics` to better convey its predictive-analytics / machine-learning focus and to read more clearly for non-technical audiences (e.g. MBA / finance). Updated the skill `name`, document titles, repository URLs, and the project directory accordingly.

### Added
- README: repository link, clone instructions, and project-status badges.

## [1.0.0] - 2026-05-26

### Added
- Initial release of the Structured Data Analysis Skill.
- Dual-format architecture: `SKILL.md` (Claude entry point with YAML header) and `METHODOLOGY.md` (plain-prompt entry point for other LLMs), sharing a common `references/` core.
- Language-adaptive output: the analysis is produced in whatever language the user asks in.
- Seven platform-neutral deep-dive guides covering the full workflow:
  1. Data import & cleaning (with data-leakage detection)
  2. Exploratory data analysis (with chart-font setup)
  3. Feature engineering
  4. Modeling (classification & regression)
  5. Evaluation, validation & diagnosis (CV, grid search, residual diagnosis, data-driven thresholds)
  6. Clustering (with PCA-based high-dimensional visualization)
  7. Interpretability, math expression & output
- Cross-cutting "analysis spirit" principles embedded throughout: leakage-first, honest reporting, baseline comparison, method-matches-constraint, data-driven thresholds, cross-validate and interpret.
- A city-noise dataset is used as the running, domain-agnostic example.
- Open-source project files: README, LICENSE (MIT), CONTRIBUTING, .gitignore.
