# Reweight to population totals estimated from a control survey

Adjusts the full-sample and replicate weights (sets of perturbed weight
columns used to compute standard errors) of `primary_design` so that its
estimated totals for `variables` match those estimated from
`control_design`. Implements the Opsomer & Erciulescu (2022) replication
variance adjustment, which correctly propagates the sampling uncertainty
of the control survey's estimates into the calibrated primary design.
When `targets` is non-NULL, additional fixed census margins are enforced
simultaneously alongside the random margins from the control survey.

## Usage

``` r
calibrate_to_survey(
  primary_design,
  control_design,
  variables,
  targets = NULL,
  type = c("prop", "count"),
  method = c("rake", "linear", "logit"),
  algorithm = c("classic_ipf", "nr"),
  bounds = c(-Inf, Inf),
  unit_scale = NULL,
  reference_design = NULL,
  control = list()
)
```

## Arguments

- primary_design:

  A `survey_replicate` or `survey_nonprob` object with replicate
  weights. The design to be calibrated. Must have replicate weights and
  a non-NULL `@variables$scale` replication constant.

- control_design:

  A `survey_replicate` or `survey_nonprob` object with replicate
  weights. The reference survey from which random population totals are
  estimated. Must have replicate weights and a non-NULL
  `@variables$scale`. Does not affect the class of the returned object.

- variables:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Bare names of the random calibration variables in
  `primary_design@data`. Must be present in both designs. These are the
  variables whose totals are estimated from the control survey and vary
  per replicate in the Opsomer algorithm.

- targets:

  `NULL` (the default) or a named list of fixed census margins. Each
  element's name is a column in `primary_design@data`. Each element is
  either a named numeric vector (names are level labels) or a tibble
  with the variable column plus `"n"` (count) or `"prop"` (proportion).
  Format A (named numeric vector) and Format B (tibble) may be mixed
  within the same list. When `NULL`, the Opsomer path runs with no fixed
  margins; `type` is matched but unused.

- type:

  `"prop"` (the default) or `"count"`. Controls interpretation of
  `targets` values. `"prop"` means proportions summing to 1.0 per
  variable (each element must sum to 1.0 within 1e-6); `"count"` means
  population counts (positive non-NA values). Ignored when
  `targets = NULL`. Matched with
  [`rlang::arg_match()`](https://rlang.r-lib.org/reference/arg_match.html).

- method:

  Calibration method: `"rake"` (the default), `"linear"`, or `"logit"`.
  Applied to both the full-sample and per-replicate calibration steps.
  Matched with
  [`rlang::arg_match()`](https://rlang.r-lib.org/reference/arg_match.html).

- algorithm:

  Raking algorithm; used only when `method = "rake"`. `"classic_ipf"`
  (the default): iterative proportional fitting (Deming & Stephan 1940)
  — the algorithm used in Opsomer & Erciulescu (2022). `"nr"`:
  Newton-Raphson raking (Deville, Sarndal & Sautory 1993). Silently
  ignored when `method != "rake"`. Matched with
  [`rlang::arg_match()`](https://rlang.r-lib.org/reference/arg_match.html).

- bounds:

  Numeric vector of length 2: `c(lower, upper)`. Bounds on the
  calibrated-to-starting-weight ratio. `c(-Inf, Inf)` (default) means no
  bounds. Passed to
  [`survey::calibrate()`](https://rdrr.io/pkg/survey/man/calibrate.html).

- unit_scale:

  `NULL` or a positive numeric vector of length
  `nrow(primary_design@data)`. Per-unit variance scaling passed as the
  `variance` argument to
  [`survey::calibrate()`](https://rdrr.io/pkg/survey/man/calibrate.html).
  `NULL` means no scaling.

- reference_design:

  A `survey_taylor` object or `NULL`. Stored in the history entry for
  provenance only; not used in computation.

- control:

  Named list of calibration control parameters. Known keys:

  - `maxit`: maximum iterations (default `50L`)

  - `epsilon`: convergence tolerance (default `1e-10`)

  Unknown keys trigger a `surveywts_warning_control_param_ignored`
  warning.

## Value

A calibrated design object whose class matches `primary_design`: a
`survey_nonprob` input returns a `survey_nonprob`; a `survey_replicate`
input returns a `survey_replicate`. Updated full-sample and replicate
weights are written back. A new entry with
`operation = "calibrate_to_survey"` is appended to the weighting
history. The history entry always includes `a_constants` (the Opsomer
perturbation constants, length `R_eff = K * R`) and `K` (the expansion
factor). When `targets` is non-NULL, the entry additionally includes
`targets` (user-supplied fixed margins), `type`, and `fixed_variables`.

## Details

**When to use.** Use
[`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md)
when your targets are fixed census values with no sampling error of
their own. Use `calibrate_to_survey()` when the targets come from
another survey: the control survey's sampling error then carries into
the replicate variance. When you have only published estimates and their
variance-covariance matrix — not the control survey's data — use
[`calibrate_to_estimate()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_estimate.md).

Both `primary_design` and `control_design` must carry replicate weights
and non-NULL `@variables$scale` replication constants. The scale
constants are required to compute the Opsomer adjustment constants
`a_r`. Use
[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md)
or another `create_*_weights()` function to create designs with scale
populated.

When `method = "linear"`, any negative calibrated full-sample weights
are clipped to `.Machine$double.eps` with a warning. Replicate weights
are never clipped.

## Algorithm

Implements the Opsomer & Erciulescu (2022) replication variance
adjustment for sample-based calibration. The core idea is to calibrate
each primary replicate to a *perturbed* version of the control-survey
totals that incorporates the control survey's replication structure.

**Notation:**

- \\R\\ = number of primary replicate columns

- \\R_C\\ = number of control replicate columns

- \\A\\ = `primary_design@variables$scale` (scalar replication constant)

- \\A_C\\ = `control_design@variables$scale`

- \\\hat{t}\_{Cx}\\ = control full-sample estimated totals for
  `variables`

- \\\hat{t}\_{Cx}^{(r)}\\ = control replicate-\\r\\ estimated totals

- \\T\_{\text{fixed}}\\ = fixed population margins from `targets` (empty
  set when `targets = NULL`)

**Step 1.** Extract \\A\\, \\A_C\\, \\R\\, \\R_C\\.

**Step 2.** Compute expansion factor: when \\R_C \> R\\, set \\K =
\lceil R_C / R \rceil\\, \\R\_{\text{eff}} = K R\\, and
\\A\_{\text{eff}} = A / K\\; otherwise \\K = 1\\, \\R\_{\text{eff}} =
R\\, \\A\_{\text{eff}} = A\\.

**Step 3.** Compute adjustment constants: \$\$a_r = \begin{cases}
\sqrt{A_C / A\_{\text{eff}}} & r = 1, \ldots, \min(R\_{\text{eff}}, R_C)
\\ 0 & r \> \min(R\_{\text{eff}}, R_C) \end{cases}\$\$ When \\K = 1\\
this reduces to \\\sqrt{A_C / A}\\.

**Step 4.** Compute full-sample control totals \\\hat{t}\_{Cx}\\ and
per-replicate control totals \\\hat{t}\_{Cx}^{(r)}\\ for each variable
in `variables`.

**Step 5.** Draw a random mapping of control replicate columns to
virtual primary replicates. One permutation is drawn per call; use
[`set.seed()`](https://rdrr.io/r/base/Random.html) before calling to
make results reproducible.

**Step 6.** Calibrate the full-sample weights of `primary_design` to the
combined target set \\\\ \hat{t}\_{Cx} \\\\ (random margins) union
\\T\_{\text{fixed}}\\ (fixed margins). This defines \\w_i^\*\\.

**Step 7.** For each primary replicate \\r = 1, \ldots, R\\: for each
virtual replicate \\s\\ in the \\K\\ repetitions of replicate \\r\\,
compute the perturbed control total: \$\$\hat{t}^\*\_{Cx}(s) =
\hat{t}\_{Cx} + a_s \bigl(\hat{t}^{(c_s)}\_{Cx} -
\hat{t}\_{Cx}\bigr)\$\$ where \\c_s\\ is the mapped control replicate
for virtual replicate \\s\\. Calibrate the *original* primary
replicate-\\r\\ weights to \\\hat{t}^\*\_{Cx}(s)\\ (random margins)
union \\T\_{\text{fixed}}\\ (fixed margins). When \\K \> 1\\, average
the \\K\\ calibrated weight vectors to obtain output replicate \\r\\.

**Step 8.** Write calibrated weights back and append history entry.

**Calibration method and algorithm:** When `method = "rake"`,
`algorithm` selects between `"classic_ipf"` (iterative proportional
fitting, Deming & Stephan 1940 — the algorithm used by Opsomer &
Erciulescu 2022) and `"nr"` (Newton-Raphson raking, Deville, Sarndal &
Sautory 1993). For `method = "linear"` or `"logit"`, `algorithm` is
matched but silently ignored.

## Convergence

Convergence failure in any `.calibrate_opsomer_single()` or
[`survey::calibrate()`](https://rdrr.io/pkg/survey/man/calibrate.html)
call raises `surveywts_error_calibration_not_converged`. A hard error in
calibration raises `surveywts_error_calibration_failed`. Users may
increase `control$maxit` or relax `control$epsilon`. Perturbed margins
that are inconsistent with fixed margins may not converge; verify that
`targets` margins are achievable given the data structure.

## Warnings

**Unknown control parameters.** Unrecognized keys in `control` are
ignored with a warning. Check spelling before assuming a key has an
effect.

**Replicate scheme mismatch.** When `primary_design` and
`control_design` have different replicate types (e.g., bootstrap vs.
jackknife), a warning is emitted. Results are still returned; the
statistical validity of mixing replicate schemes is the user's
responsibility.

**Negative calibrated weights.** When `method = "linear"` produces
negative calibrated full-sample weights, those weights are clipped to
`.Machine$double.eps` and a warning is emitted. Replicate weights are
never clipped.

## Limitations

**Independence assumption.** The Opsomer & Erciulescu (2022) consistency
proof assumes `primary_design` and `control_design` are independent
samples from non-overlapping draws. Designs that share respondents
(e.g., panel waves, subsamples of the control, or any linked-sample
design) violate this assumption. The function cannot detect overlap;
users are responsible for verifying the independence condition.
Overlapping samples produce variance estimates that do not account for
the covariance between the two surveys' totals.

**Nonprobability samples.** The Opsomer & Erciulescu (2022) consistency
proof assumes probability samples with design-based replicate variance.
When `survey_nonprob` designs are supplied, the method operates
mechanically but the variance interpretation depends on how the
replicate weights were constructed.

## References

Opsomer, J.D. and Erciulescu, A.L. (2022). Replication variance
estimation after sample-based calibration. *Survey Methodology*, 47(2),
265–277.

Fuller, W.A. (1998). Replication variance estimation for two-phase
samples. *Statistica Sinica*, 8, 1153–1164.

## See also

[`calibrate_to_estimate()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_estimate.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other sample-calibration:
[`calibrate_to_estimate()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_estimate.md)

## Examples

``` r
# calibrate ns_wave1 NPS to NPORS probability survey on age_f3 and sex ----

# calibrate the non-probability sample first; the quasi-randomization
# bootstrap replays that step inside every replicate
edu_targets <- c(
  "Less than HS" = 0.09, "HS/Some college" = 0.55, "College+" = 0.36
)
ns_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)
ns_cal <- calibrate_rake(ns_svy, targets = list(edu_f3 = edu_targets))

# `type = "quasi-randomization"` is required here. The default type wraps a
# non-probability sample as a simple random sample and returns a
# `survey_replicate`, which this function rejects. See the Getting started
# article. Real analyses use more replicates; 10 keeps `R CMD check` fast.
primary <- create_bootstrap_weights(
  ns_cal, type = "quasi-randomization", replicates = 10L, seed = 1L
)

npors_design <- surveycore::as_survey(
  npors_2025_clean, weights = weight, strata = stratum
)
control <- create_bootstrap_weights(npors_design, replicates = 20L, seed = 1L)

# the replicate mapping between the two designs is drawn at random
set.seed(1)
result <- calibrate_to_survey(primary, control, variables = c(age_f3, sex))

summarize_weights(result)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0 0.752  1.38 0.00204 0.112 0.295 0.849  4.76 2202.
```
