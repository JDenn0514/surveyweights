# Tools for Survey Weighting and Calibration

Provides the full weight adjustment workflow for survey data. Calibrates
weights to known population totals by raking, linear (GREG),
logit-bounded, or post-stratification estimators. Corrects unit
nonresponse, estimates inverse probability weights for a non-probability
sample, and generates replicate weights for variance estimation. Trims
and rescales weights, reports weight diagnostics, and records every
adjustment in the weighting history of the survey object.

## Key Functions

**Calibration:**

- [`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md):
  a thin dispatcher that routes to
  [`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
  [`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md),
  or
  [`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md)
  based on `method`

- [`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md):
  linear (GREG) calibration to marginal totals; `bounds` switches it to
  truncated-linear calibration

- [`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md):
  logit-bounded calibration; the g-weights stay inside an open interval,
  so the calibrated weights stay positive

- [`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md):
  raking (iterative proportional fitting) to several marginal totals at
  the same time

- [`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md):
  matches the exact cross-tabulation cells in one pass instead of the
  marginal totals

**Sample-Based Calibration:**

- [`calibrate_to_survey()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_survey.md):
  reweights a design to the totals estimated from a control survey, and
  propagates the uncertainty of those estimates

- [`calibrate_to_estimate()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_estimate.md):
  reweights a design to externally supplied count totals, with a
  variance-covariance matrix for those totals

**Propensity Weighting:**

- [`ipw()`](https://jdenn0514.github.io/surveywts/reference/ipw.md):
  estimates inverse probability weights for a non-probability sample
  from the participation propensity against a reference sample

**Nonresponse Adjustment:**

- [`adjust_nonresponse()`](https://jdenn0514.github.io/surveywts/reference/adjust_nonresponse.md):
  moves the weight of the nonrespondents to the respondents inside
  weighting classes

- [`redistribute_weights()`](https://jdenn0514.github.io/surveywts/reference/redistribute_weights.md):
  sets the weight of the excluded rows to zero and moves that weight to
  the retained rows

**Replicate Weights:**

- [`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md):
  a dispatcher that routes to one of the `create_*_weights()` functions
  based on `method`

- [`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md):
  bootstrap replicate weights, or quasi-randomization bootstrap weights
  for a non-probability sample

- [`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md):
  jackknife replicate weights, including the delete-a-group jackknife
  for a non-probability sample

- [`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md):
  balanced repeated replication (BRR) or Fay's BRR weights; the design
  must have exactly 2 PSUs per stratum

- [`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md):
  successive difference replication weights

- [`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md):
  generalized bootstrap replicate weights for a given target variance
  estimator

- [`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md):
  Fay's generalized replication weights, which are deterministic

- [`as_taylor_design()`](https://jdenn0514.github.io/surveywts/reference/as_taylor_design.md):
  converts a replicate design back to a Taylor design and drops the
  replicate weight columns

**Weight Utilities:**

- [`trim_weights()`](https://jdenn0514.github.io/surveywts/reference/trim_weights.md):
  clips the weights to a bounded interval and spreads the trimmed excess
  across the untrimmed units

- [`rescale_weights()`](https://jdenn0514.github.io/surveywts/reference/rescale_weights.md):
  rescales the weights to sum to the sample size, either overall or
  within each group

**Diagnostics:**

- [`effective_sample_size()`](https://jdenn0514.github.io/surveywts/reference/effective_sample_size.md):
  Kish's effective sample size

- [`weight_variability()`](https://jdenn0514.github.io/surveywts/reference/weight_variability.md):
  the coefficient of variation of the weights

- [`summarize_weights()`](https://jdenn0514.github.io/surveywts/reference/summarize_weights.md):
  a table of weight distribution statistics, optionally by group

## See also

Useful links:

- <https://github.com/JDenn0514/surveywts>

- <https://jdenn0514.github.io/surveywts/>

- Report bugs at <https://github.com/JDenn0514/surveywts/issues>

## Author

**Maintainer**: Jacob Dennen <jdenn0514@gmail.com>
([ORCID](https://orcid.org/0000-0003-3006-7364)) \[copyright holder\]

Authors:

- Jacob Dennen <jdenn0514@gmail.com>
  ([ORCID](https://orcid.org/0000-0003-3006-7364)) \[copyright holder\]
