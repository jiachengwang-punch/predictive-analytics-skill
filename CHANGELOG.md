# Changelog

All notable changes to this skill are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/), and this project adheres to
semantic versioning.

## [1.0.0] - 2026-05-26

### Added
- Initial release of the Predictive Analytics Skill.
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
