# Contributing

Thanks for your interest in improving this skill. Contributions of all kinds are welcome — fixing typos, clarifying a guide, adding a new reference, or improving the methodology.

## What you can contribute

- **Improve the guides.** Clarify explanations, fix errors, add better code patterns or worked examples in `references/`.
- **Extend the methodology.** Add a new stage guide (e.g. time-series forecasting, anomaly detection) following the existing format.
- **Improve triggering.** Refine the `description` in `SKILL.md` so the skill activates on the right requests (keep it under 1024 characters).
- **Add examples.** Worked examples on additional domains (churn, demand, risk, etc.) strengthen the running-example coverage.
- **Translations.** The methodology is language-adaptive at runtime, but documentation contributions in other languages are welcome.

## Guidelines

### Keep the dual-format architecture in sync
This skill has two entry points (`SKILL.md` for Claude, `METHODOLOGY.md` for other LLMs) that share the `references/` core. If you change the methodology, the workflow, or the cross-cutting principles, **update both entry points** so they stay consistent.

### Respect the structure
- `SKILL.md` should stay concise (ideally under ~500 lines) and act as a navigator to the references.
- Each `references/*.md` guide is self-contained, platform-neutral, and explains the *why* behind each step — not just the *how*.
- For long guides (>300 lines), include a table of contents.

### Preserve the analysis spirit
The value of this skill is its rigor. When editing, keep the core principles intact: leakage detection first, honest reporting, baseline comparison, method-matches-constraint, data-driven thresholds, cross-validate and interpret.

### Code patterns
Code examples target Python with pandas, scikit-learn, lightgbm, shap, matplotlib, seaborn, scipy. Keep examples adaptable (use placeholder column names), not hard-coded to one dataset.

## How to submit

1. Fork the repository and create a branch for your change.
2. Make your edits; if you touched the methodology, update both `SKILL.md` and `METHODOLOGY.md`.
3. If you changed `SKILL.md`'s `description`, verify it's under 1024 characters.
4. Open a pull request with a clear description of what changed and why.

## Reporting issues

Open an issue describing the problem or suggestion. For methodology questions, include the dataset type and the stage you're working on so the discussion has context.
