# Package index

## Calibration

Functions for calibrating survey weights to known population totals.

- [`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md)
  : Adjust weights to match population totals
- [`calibrate_linear()`](https://jdenn0514.github.io/surveywts/reference/calibrate_linear.md)
  : Fit weights using linear (GREG) calibration
- [`calibrate_logit()`](https://jdenn0514.github.io/surveywts/reference/calibrate_logit.md)
  : Fit weights using logit-bounded calibration
- [`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md)
  : Fit weights using raking
- [`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md)
  : Fit weights using post-stratification

## Sample-Based Calibration

Functions for calibrating replicate-weight designs to a control survey
or to estimated totals with uncertainty.

- [`calibrate_to_survey()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_survey.md)
  : Reweight to population totals estimated from a control survey
- [`calibrate_to_estimate()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_estimate.md)
  : Reweight to externally estimated population totals

## Propensity Weighting

Functions for inverse probability weighting of non-probability samples.

- [`ipw()`](https://jdenn0514.github.io/surveywts/reference/ipw.md) :
  Estimate inverse probability weights for a non-probability sample

## Example Datasets

Bundled datasets for IPW examples and calibration examples.

### Non-probability sample

Bundled datasets for IPW examples and calibration examples.

- [`ns_wave1`](https://jdenn0514.github.io/surveywts/reference/ns_wave1.md)
  : National Survey Wave 1 with harmonized demographic columns

### Probability reference surveys

- [`cps_2023`](https://jdenn0514.github.io/surveywts/reference/cps_2023.md)
  : CPS ASEC 2023 national adult sample with harmonized demographic
  columns
- [`gss_2024`](https://jdenn0514.github.io/surveywts/reference/gss_2024.md)
  : GSS 2024 sample data with harmonized demographic columns
- [`npors_2025`](https://jdenn0514.github.io/surveywts/reference/npors_2025.md)
  : Pew NPORS 2025 probability sample with harmonized demographic
  columns
- [`npors_2025_clean`](https://jdenn0514.github.io/surveywts/reference/npors_2025_clean.md)
  : Pew NPORS 2025 complete cases only

### Calibration example data

- [`pew_2016_optin`](https://jdenn0514.github.io/surveywts/reference/pew_2016_optin.md)
  : Pew 2016 ATP opt-in sample
- [`pew_2016_synth_pop`](https://jdenn0514.github.io/surveywts/reference/pew_2016_synth_pop.md)
  : Pew 2016 ATP synthetic population

## Nonresponse Adjustment

Functions for adjusting survey weights for unit nonresponse.

- [`adjust_nonresponse()`](https://jdenn0514.github.io/surveywts/reference/adjust_nonresponse.md)
  : Correct weights for unit nonresponse
- [`redistribute_weights()`](https://jdenn0514.github.io/surveywts/reference/redistribute_weights.md)
  : Transfer weight from excluded rows to retained rows

## Replicate Weights

Functions for creating replicate weights for variance estimation.

- [`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md)
  : Generate bootstrap replicate weights
- [`create_jackknife_weights()`](https://jdenn0514.github.io/surveywts/reference/create_jackknife_weights.md)
  : Construct jackknife replicate weights
- [`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md)
  : Generate BRR (Fay) replicate weights
- [`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md)
  : Generate generalized bootstrap replicate weights
- [`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md)
  : Generate generalized replication replicate weights
- [`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md)
  : Generate successive difference replication weights
- [`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md)
  : Generate replicate weights for a survey design
- [`as_taylor_design()`](https://jdenn0514.github.io/surveywts/reference/as_taylor_design.md)
  : Convert a replicate design back to a Taylor design

## Weight Utilities

Functions for trimming and rescaling survey weights.

- [`trim_weights()`](https://jdenn0514.github.io/surveywts/reference/trim_weights.md)
  : Clip weights to a bounded interval
- [`rescale_weights()`](https://jdenn0514.github.io/surveywts/reference/rescale_weights.md)
  : Rescale survey weights to a target mean or sum

## Diagnostics

Functions for assessing the distribution and quality of survey weights.

- [`effective_sample_size()`](https://jdenn0514.github.io/surveywts/reference/effective_sample_size.md)
  : Estimate Kish's effective sample size of weighted data
- [`weight_variability()`](https://jdenn0514.github.io/surveywts/reference/weight_variability.md)
  : Measure how unequal the survey weights are
- [`summarize_weights()`](https://jdenn0514.github.io/surveywts/reference/summarize_weights.md)
  : Report summary statistics for the weight distribution
