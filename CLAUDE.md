# surveywts Package Development

**Part of the surveyverse ecosystem.**

surveywts provides tools for survey weighting and calibration.

------------------------------------------------------------------------

## Release Status

| Release | Tag | Status | Notes |
|----|----|----|----|
| Calibration | `v0.1.0` | ✅ Complete | [`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md), [`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md), [`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md), [`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md), [`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md), basic diagnostics |
| Replicate | minor bump | ✅ Complete | All `create_*_weights()` functions; [`as_taylor_design()`](https://jdenn0514.github.io/surveywts/reference/as_taylor_design.md) |
| Utilities | minor bump | ✅ Complete | [`trim_weights()`](https://jdenn0514.github.io/surveywts/reference/trim_weights.md), [`rescale_weights()`](https://jdenn0514.github.io/surveywts/reference/rescale_weights.md) |
| Nonresponse | minor bump | ✅ Complete | [`calibrate_to_survey()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_survey.md), [`calibrate_to_estimate()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_estimate.md), [`adjust_nonresponse()`](https://jdenn0514.github.io/surveywts/reference/adjust_nonresponse.md), [`redistribute_weights()`](https://jdenn0514.github.io/surveywts/reference/redistribute_weights.md) |
| Propensity | minor bump | ✅ Complete | Non-probability sample IPW; unlocks propensity nonresponse |
| Diagnostics | minor bump | 🔜 Next | Balance assessment, `check_balance()`, `diagnose_propensity()`, `compare_weighted_estimates()` |
| Polish | minor bump | ⬜ Pending | Vignettes, `--as-cran` clean, pkgdown |

Full roadmap at `plans/roadmap.md`.

------------------------------------------------------------------------

## Where Things Live

- `plans/error-messages.md` — canonical error/warning class names and
  CLI message templates
- `.claude/WORKFLOW.md` — how the skills fit together (planning arc →
  implementation loop)
- `.claude/rules/core.md` — cross-role rules that auto-load; its pointer
  table names the one standards file for each concern
- `.claude/standards/` — the 8 role-scoped standards files (code style,
  documentation, package conventions, testing, GitHub strategy,
  engineering preferences); read on demand, not auto-loaded

## Before You Write or Review R Code

Read the standards file for what you are writing or reviewing. The
pointer table in `.claude/rules/core.md` names which one.
