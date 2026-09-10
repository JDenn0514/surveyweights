# Estimate inverse probability weights for a non-probability sample

Constructs inverse probability weights for a non-probability sample
(NPS) by estimating participation propensity via pseudo-likelihood
logistic regression. The weights adjust for selection bias by
upweighting NPS units that are underrepresented relative to a
probability-based reference sample. This assumes missing at random
(response depends only on observed variables): participation may depend
on the covariates in `selection`, but not on the outcome itself. Routes
to the MLE Newton-Raphson path or the GEE calibration path based on
`estimating_eq`; see the **Algorithm** section for method details.

## Usage

``` r
ipw(
  data,
  reference,
  selection = NULL,
  predictors = NULL,
  missing_method = c("omit", "separate", "impute"),
  mice_args = list(),
  method = "logit",
  estimating_eq = c("gee", "mle"),
  maxit = 25L,
  epsilon = 1e-08,
  adjust_reference = TRUE,
  trim = FALSE,
  population_size = NULL,
  wt_name = "ipw_weight"
)
```

## Arguments

- data:

  A `data.frame` containing the non-probability sample.

- reference:

  A `survey_taylor` or `survey_replicate` object representing the
  probability-based reference sample. Must have strictly positive design
  weights. When a `survey_replicate` is supplied, only the main design
  weights (`@variables$weights`) are used for propensity estimation; the
  replicate weight columns are not used by `ipw()`. See the
  **Limitations** section for guidance on reference sample quality and
  covariate measurement requirements.

- selection:

  A one-sided formula (e.g., `~ age + sex`) specifying the covariates
  used to model participation propensity. Exactly one of `selection` and
  `predictors` must be supplied.

- predictors:

  A character vector of covariate names (e.g., `c("age", "sex")`), used
  as an alternative to `selection`. Exactly one of `selection` and
  `predictors` must be supplied. Suitable for programmatic use (e.g.,
  [`lapply()`](https://rdrr.io/r/base/lapply.html)).

- missing_method:

  `"omit"` (the default), `"separate"`, or `"impute"`. Controls how `NA`
  values in `selection` variables in `data` are handled. See the
  **Missing Data** section for full documentation of each option.

- mice_args:

  Named list of additional arguments forwarded to
  [`mice::mice()`](https://amices.org/mice/reference/mice.html) when
  `missing_method = "impute"`. The argument `m` is fixed at `1` and
  cannot be overridden — attempting to do so triggers
  `surveywts_warning_ipw_mice_m_ignored`. Ignored when `missing_method`
  is not `"impute"`.

- method:

  `"logit"` (the default), `"probit"`, or `"cloglog"`. Link function for
  the propensity model. Partial matching is supported.

- estimating_eq:

  `"gee"` (the default) or `"mle"`. Estimating equation for the
  propensity model. Partial matching is supported.

  - `"gee"` uses calibration estimating equations that guarantee
    \\\sum_k w_k x_k = \sum_k d_k x_k\\ at convergence, where \\w_k\\
    are the IPW weights, \\d_k\\ are reference design weights, and
    \\x_k\\ are the covariate values. When `adjust_reference = TRUE` and
    `nps_fraction > 0.05`, the calibration target uses Valliant-adjusted
    reference totals. When `missing_method = "separate"`, the guarantee
    applies to complete-case NPS rows only.

  - `"mle"` uses the pseudo-likelihood score equation. Weights reproduce
    the reference-weighted covariate totals in expectation but not
    exactly.

- maxit:

  Maximum iterations for the propensity solver. For MLE, the maximum
  number of Newton-Raphson steps. For GEE, passed as `control$maxit` to
  [`nleqslv::nleqslv()`](https://bertcarnell.github.io/nleqslv/reference/nleqslv.html)
  (controls the solver's internal iteration budget). Must be \>= 1.
  Default `25L`.

- epsilon:

  Convergence threshold. For MLE, applied to the maximum absolute
  Newton-Raphson step size. For GEE, passed as both `xtol` and `ftol` to
  [`nleqslv::nleqslv()`](https://bertcarnell.github.io/nleqslv/reference/nleqslv.html);
  convergence is declared when either the step-size criterion or the
  score-norm criterion is satisfied. Must be \> 0. Default `1e-8`.

- adjust_reference:

  Logical (default `TRUE`). Whether to apply Valliant (2020) Eq. (1)
  reference weight adjustment when the NPS is a non-negligible fraction
  of the estimated population. When `nps_fraction = nrow(data) / sum(d)`
  (where `d` are the reference design weights after excluding rows with
  `NA` in any selection variable) exceeds 0.05, the reference weights
  are multiplied by `(N_hat - n_NPS) / N_hat` to prevent the
  reference-side score denominator from being inflated by NPS units
  already counted on the NPS side. When `nps_fraction <= 0.05`, no
  adjustment is applied regardless of this argument. Set
  `adjust_reference = FALSE` to skip the adjustment when the NPS and
  reference frames are known to be disjoint and the correction is
  inappropriate.

- trim:

  Logical. If `TRUE`, IPW weights are trimmed at
  `median(w) + 5 * IQR(w)` after estimation. Default `FALSE`.

- population_size:

  Optional positive numeric scalar. If the population size N is known
  from a census or frame, supply it here. When provided,
  `estimated_population_size` in the history entry records this known
  value and `population_size_known` is set to `TRUE`. When `NULL`
  (default), `population_size_known = FALSE` and the self-normalizing
  estimate `N_hat = sum(1 / pi_hat)` is recorded (IPW2/Hájek). This
  value is stored for reference only and does not affect the returned
  weights.

- wt_name:

  Name for the output weight column in `@data`. Must be a non-empty
  character scalar that does not already exist in `data`. Default
  `"ipw_weight"`.

## Value

A `survey_nonprob` object. The `@data` slot contains all original
columns from `data` plus a new column named `wt_name` holding the
(possibly trimmed) IPW weights. When `missing_method = "omit"`, rows
with NA in any selection variable are excluded from `@data`. The
`@reference_sample` slot holds the supplied `reference` design. A new
entry with `operation = "ipw"` is appended to the weighting history. The
`estimated_population_size` field in the history entry always equals
`sum(w)` before trimming, regardless of `trim`.

## Details

**Variance estimation — refit required:** Naive variance estimates from
the returned `survey_nonprob` object treat the propensity scores (the
modeled probability of responding or of appearing in the sample) as
fixed and underestimate variance. Correct variance estimation requires a
replication approach in which the propensity model is **refit at every
replicate** (bootstrap resample or jackknife group) so that estimation
uncertainty in the propensity parameters is captured (Elliott &
Valliant, 2017; Valliant, 2020).

Jackknife is generally preferred over bootstrap when point estimates are
nearly unbiased: Valliant (2020, Table 9) shows jackknife confidence
interval coverage consistently nearer the 95% nominal level than
bootstrap with replacement. Both require refitting the propensity model
at each replicate.

A correct jackknife procedure (preferred):

1.  Partition `data` into G groups (G = 20 is a standard choice;
    Valliant (2020, §2.1.4) uses G = 20 in simulations).

2.  For each group g (1..G), omit group g from `data` and call `ipw()`
    on the reduced NPS dataset and the full `reference`.

3.  Compute the estimand from each of the G replicate weighted samples.

4.  Compute jackknife variance:
    `V_JK = ((G - 1) / G) * sum((theta_g - mean(theta_g))^2)`.

A correct bootstrap procedure (valid alternative):

1.  Resample `data` with replacement (simple random, or cluster-aware if
    the NPS has a known cluster structure).

2.  Resample `reference` using a design-respecting method — for complex
    designs, use Rao-Wu rescaled bootstrap
    (`survey::as.svrepdesign(type = "subbootstrap")`) rather than plain
    SRS resampling.

3.  Call `ipw()` on each resample pair to produce new weights.

4.  Compute the estimand from each replicate's weighted sample.

5.  Use replicate variance as the variance estimate.

Both procedures apply equally when `estimating_eq = "gee"`. Variance
estimates that do not refit the propensity model at each replicate will
be anti-conservative.

**`survey_replicate` reference:** When `reference` is a
`survey_replicate` object, `ipw()` uses only the main design weights
(`reference@variables$weights`) for propensity estimation. The replicate
weight columns stored in the `survey_replicate` object are not used —
variance estimation still requires refitting the propensity model at
each replicate as described above. The population size estimate
\\\hat{N}\_p\\ is computed from the main weights in the usual way (Wu,
2022, §6.2).

**Estimating equation:** `ipw()` uses the *unconditional*
pseudo-likelihood approach (Valliant & Dever, 2011, as described in
Elliott & Valliant, 2017, p. 256). NPS units enter the score equation
with implicit weight 1; reference units enter with their design weights.
This estimates P(NPS \| in population) directly, rather than P(NPS \| in
combined sample). The alternative *conditional* approach — pooling both
samples and running an unweighted logistic regression with NPS
membership as the outcome — is not used because it does not account for
reference design weights and estimates a different quantity (Chen, Li &
Wu, 2020, §2.1).

**Doubly robust estimation (recommended):** The papers in `@references`
unanimously recommend combining IPW weights with an outcome regression
model to form a doubly robust (DR) estimator. A DR estimator is
consistent if *either* the propensity model *or* the outcome regression
model is correctly specified — providing protection against
misspecification of either. Valliant (2020) found DR "was the best
combination in this study in terms of bias, RMSE, and confidence
interval coverage"; Yang et al. (2020) show that DR substantially
outperforms IPW-only under propensity misspecification. The weights
returned by `ipw()` are the propensity component of such a DR pipeline;
the outcome regression step is planned for a future release.

**Quantile balancing approximation:** Beresewicz et al. (2025) show that
augmenting the propensity model with quantile-indicator variables for
continuous covariates substantially reduces bias under nonlinear
selection ("quantile balancing IPW"). Users can approximate this by
adding cut-point indicators to `selection`:

    nps$age_q <- cut(
      nps$age,
      quantile(nps$age, c(0, .25, .5, .75, 1)),
      include.lowest = TRUE
    )
    ipw(nps, ref, selection = ~age_q + sex)

Native QBIPW support (Beresewicz et al., 2025, eqs. 4.1–4.2) is planned
for a future release.

## Algorithm

**MLE** — pseudo-likelihood score equation (Chen, Li & Wu, 2020;
Beresewicz et al., 2025, eq. 3.1):

\$\$U\_{MLE}(\gamma) = \sum\_{k \in NPS} x_k - \sum\_{k \in ref} d_k
\pi_k(\gamma) x_k = 0\$\$

where \\\pi_k(\gamma) = \text{link}^{-1}(x_k^\top \gamma)\\ is the
propensity score under link function `method`. The system is solved by
Newton-Raphson with step \\\gamma\_{t+1} = \gamma_t - H^{-1} U\\, where
\\H = -X\_{ref}^\top \text{diag}(d \pi(1-\pi)) X\_{ref}\\ is the
Hessian. IPW weights are \\w_k = 1 / \pi_k(\hat\gamma)\\ — the
reciprocal of the estimated participation propensity. At convergence,
the weighted NPS covariate totals match the reference totals in
expectation.

**GEE** — calibration estimating equations (Beresewicz et al., 2025, eq.
3.3):

\$\$U\_{GEE}(\gamma) = \sum\_{k \in NPS} \frac{x_k}{\pi_k(\gamma)} -
\sum\_{k \in ref} d_k x_k = 0\$\$

At convergence, this guarantees exact covariate balance: \\\sum_k w_k
x_k = \sum_k d_k x_k\\. The system is solved by
[`nleqslv::nleqslv()`](https://bertcarnell.github.io/nleqslv/reference/nleqslv.html)
with the analytical Jacobian \\J(\gamma) = -X\_{NPS}^\top
\text{diag}((1-\pi)/\pi) X\_{NPS}\\ and Newton method with double-dogleg
global strategy. Convergence is declared when the nleqslv termination
code is 1 (function criterion near zero) or 2 (step criterion near
zero). IPW weights are \\w_k = 1 / \pi_k(\hat\gamma)\\.

## Convergence

**MLE:** Convergence is declared when \\\max_j \|\delta_j\| \<
\epsilon\\, where \\\delta\\ is the Newton-Raphson step vector. If the
MLE algorithm does not converge within `maxit` iterations, a warning is
issued with class `surveywts_warning_propensity_nr_no_convergence` and
the result from the last iteration is returned. The convergence
diagnostic reported in the warning message is `max(abs(delta))`.

**GEE:** Convergence is delegated to
[`nleqslv::nleqslv()`](https://bertcarnell.github.io/nleqslv/reference/nleqslv.html).
Convergence is declared when nleqslv returns termination code 1
(function-norm criterion satisfied) or 2 (step-size criterion
satisfied). Both criteria use the `epsilon` threshold. If the GEE solver
does not converge (`termcd >= 3`), the same
`surveywts_warning_propensity_nr_no_convergence` warning is issued and
scores from the last iterate are returned. The convergence diagnostic
reported in the warning message is `max(abs(fvec))`, the maximum
absolute score residual at the final iterate.

## Missing Data

**Reference sample:** NA values in `reference@data` selection variables
are always handled by listwise deletion. Rows with any NA in a selection
variable are dropped before fitting, and a warning reports the count and
which variables.

**NPS sample** (`missing_method` controls):

- `"omit"` (default): Rows with any NA in a selection variable are
  dropped before fitting and excluded from `@data`. A warning reports
  the count and which variables.

- `"separate"`: NA values in **factor and character** selection
  variables are recoded to an explicit `"(Missing)"` baseline category
  so those units still enter the propensity model and receive weights.
  Numeric selection variables with NA are not supported — `ipw()` errors
  with `surveywts_error_separate_numeric_na`; convert to a factor (e.g.,
  with [`cut()`](https://rdrr.io/r/base/cut.html)) or use `"impute"`
  instead. The propensity model is fitted on complete-case NPS rows
  only; scores are predicted for all rows by substituting `"(Missing)"`
  with the reference baseline level. This adaptation has no published
  theoretical validation; use `"impute"` for a more principled approach.

- `"impute"`: Missing values are imputed via a single iteration of
  predictive mean matching using
  [`mice::mice()`](https://amices.org/mice/reference/mice.html)
  (requires the `mice` package). All NPS units receive weights.

## Limitations

**Reference sample quality:** The reference sample must represent the
target population without material coverage or nonresponse bias — design
weights alone do not correct for an internally biased reference survey.
Elliott & Valliant (2017) recommend using large, well-controlled
probability surveys (e.g., government-conducted household surveys) as
the reference; a biased reference will produce biased propensity
estimates regardless of model specification.

**Covariate measurement:** Shared covariates must be measured with the
same question wording, response options, and measurement period in both
samples — category differences (e.g., 4-point vs. 5-point scales)
produce spurious covariate imbalance that the propensity model cannot
correct (Valliant, 2020).

**Missing at random (MAR) assumption:** `ipw()` is consistent only if
NPS participation is independent of the outcome variable given the
observed covariates in `selection` — formally, \\P(I\_{NPS} = 1 \| X, Y)
= P(I\_{NPS} = 1 \| X)\\. This is called MAR or "non-informative
sampling" (Chen, Li & Wu, 2020, Assumption A1; Valliant, 2020). It is
not testable from observed data.

**Common support:** IPW requires that every covariate combination
observed in the NPS is also present in the reference. If not, scores
approach 1 and weights become uninformative.

**Independence of participation:** The pseudo-likelihood assumes NPS
participation decisions are independent across units given the
covariates in `selection` (Chen, Li & Wu, 2020, Assumption A3). This
assumption fails when NPS units are clustered (household panels,
snowball recruitment). In clustered NPS settings the propensity model
should include cluster-level covariates, and variance estimation should
use cluster-aware resampling.

**Non-overlapping samples:** The pseudo-likelihood codes NPS units as
members (1) and reference units as non-members (0). If the same
individual appears in both `data` and `reference`, the coding is
inconsistent and propensity estimates will be biased. Remove any units
present in both samples from `reference` before calling `ipw()`
(Valliant, 2020, §2.1.2).

**Sensitivity to propensity model misspecification:** IPW estimates can
be severely biased when selection is nonlinear and the `selection`
formula does not capture this nonlinearity. Chen, Li & Wu (2020,
Table 1) demonstrate ~25% relative bias under a misspecified propensity
model even when the correct variables are included. Beresewicz et al.
(2025) show RMSE increases of 30x or more under nonlinear selection. To
mitigate this risk: add interaction or polynomial terms to `selection`
if nonlinear selection is suspected; follow `ipw()` with a doubly robust
step; or use `diagnose_propensity()` (planned) to assess covariate
balance.

**High-dimensional selection:** For large covariate vectors, Elliott &
Valliant (2017) recommend regularized alternatives such as
LASSO-penalized logistic regression, BART, or super learner. `ipw()`
uses an unpenalized propensity model and may overfit in high-dimensional
settings. In such cases, fit the propensity model externally, extract
predicted probabilities, and use them directly.

## Warnings

A convergence warning is issued when the propensity solver does not
reach the `epsilon` threshold within `maxit` iterations. The warning
reports a convergence diagnostic and suggests increasing `maxit`,
relaxing `epsilon`, or checking for extreme covariate imbalance. Scores
from the last iterate are returned; inspect them before use.

## References

Elliott, M.R. and Valliant, R. (2017). Inference for nonprobability
samples. *Statistical Science* **32**(2), 249–264.

Chen, Y., Li, P. and Wu, C. (2020). Doubly robust inference with
nonprobability survey samples. *Journal of the American Statistical
Association* **115**(532), 2011–2021.

Lenau, S., Marchetti, S., Munnich, R., Pratesi, M., Salvati, N., Shlomo,
N., Schirripa Spagnolo, F. and Zhang, L.-C. (2021). Methods for sampling
and inference with non-probability samples. Deliverable D11.8, InGRID-2
project 730998 – H2020.

Valliant, R. and Dever, J.A. (2011). Estimating propensity adjustments
for volunteer web surveys. *Sociological Methods & Research* **40**(1),
105–137.

Valliant, R. (2020). Comparing alternatives for estimation from
nonprobability samples. *Survey Methods: Insights from the Field*
**16**(1).
[doi:10.13094/SMIF-2020-00011](https://doi.org/10.13094/SMIF-2020-00011)

Yang, S., Kim, J.K. and Song, R. (2020). Doubly robust inference when
combining probability and non-probability samples with high dimensional
data. *Journal of the Royal Statistical Society: Series B* **82**(2),
445–465.

Beresewicz, M., Szymkowiak, M. and Chlebicki, P. (2025). Quantile
balancing inverse probability weighting for non-probability samples.
*arXiv preprint* arXiv:2403.09028.

Wu, C. (2022). *Statistical inference with non-probability survey
samples*. *Survey Methodology* **48**(2), 283–311.

## See also

[`adjust_nonresponse()`](https://jdenn0514.github.io/surveywts/reference/adjust_nonresponse.md)
for unit nonresponse adjustment via weighting class methods, which can
serve as the IPW step in a doubly robust pipeline when the nonresponse
mechanism is modeled.

[`calibrate_to_survey()`](https://jdenn0514.github.io/surveywts/reference/calibrate_to_survey.md)
for post-stratification and raking calibration that can be applied after
`ipw()` as the regression correction step of a doubly robust estimator.

[`summarize_weights()`](https://jdenn0514.github.io/surveywts/reference/summarize_weights.md)
for diagnosing the distribution of the IPW weights produced by this
function.

`diagnose_propensity()` (planned) for propensity score diagnostics
including AUC, covariate balance plots, and standardized mean
differences. Uses the `propensity_scores` stored in the history entry
returned by `ipw()` without refitting the model.

For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

## Examples

``` r
# --- GSS 2024 as probability reference, both interfaces ---------------
# ipw() always listwise-deletes reference rows with NA in a selection
# variable, and warns once per call. Drop them first, as the warning says.
gss_complete <- gss_2024[!is.na(gss_2024$sex) & !is.na(gss_2024$age_f3), ]
gss_ref <- surveycore::as_survey(
  gss_complete, weights = wt_pop, strata = vstrat, ids = vpsu, nest = TRUE
)
result1 <- ipw(ns_wave1, gss_ref, selection = ~sex + age_f3)
result2 <- ipw(ns_wave1, gss_ref, predictors = c("sex", "age_f3"))
summarize_weights(result1)
#> # A tibble: 1 × 11
#>       n n_positive n_zero   mean    cv    min    p25    p50    p75    max   ess
#>   <int>      <int>  <int>  <dbl> <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl> <dbl>
#> 1  6422       6422      0 39063. 0.174 30526. 33434. 37342. 45156. 50435. 6233.
calibrate(result2, list(sex = c(Male = 0.49, Female = 0.51)))
#> ℹ Raking converged in 1 sweep: all variables already met their margins. Weights
#>   were not adjusted.
#> # A calibrated survey design: 6,422 observations, 186 variables
#> # Variance: model-assisted (SRS assumption)
#> # IDs: ~1 | Strata: NULL | Weights: ipw_weight 
#> # Weighting history: 2 steps 
#> #   Step 1 [2026-09-10]: ipw [~sex + age_f3, logit, n_ref=3197, N_hat=250865240] 
#> #   Step 2 [2026-09-10]: raking (targets: sex) 

# --- CPS ASEC 2023 as survey_replicate reference -----------------------
ns_complete <- ns_wave1[!is.na(ns_wave1$race_f4), ]
cps_ref <- surveycore::as_survey_replicate(
  cps_2023, weights = "wtfinl", repweights = paste0("repwtp", 1:160),
  type = "successive-difference", scale = 4 / 160, rscales = rep(1, 160)
)
result_cps <- ipw(
  ns_complete, cps_ref, selection = ~sex + age_f3 + race_f4 + edu_f3
)
summarize_weights(result_cps)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6302       6302      0 3000. 0.247 1463. 2521. 2840. 3492. 6321. 5939.

# --- Pew NPORS 2025: the three missing_method values -------------------
npors_ref <- surveycore::as_survey(
  npors_2025_clean, weights = wt_pop, strata = stratum
)
sel <- ~sex + age_f3 + race_f4 + edu_f3
result_omit <- ipw(ns_wave1, npors_ref, selection = sel)
#> Warning: ! 120 row(s) in `data` dropped: NA in race_f4.
#> ℹ Rows with any NA in a `selection` variable are excluded when `missing_method
#>   = "omit"`.
#> ✔ Use `missing_method = "separate"` or `missing_method = "impute"` to retain
#>   rows with NA.
#> ! 120 row(s) in `data` dropped: NA in race_f4.
result_sep <- ipw(
  ns_wave1, npors_ref, selection = sel, missing_method = "separate"
)
nrow(result_sep@data) - nrow(result_omit@data)
#> [1] 120
if (requireNamespace("mice", quietly = TRUE)) {
  result_imp <- ipw(
    ns_wave1, npors_ref, selection = sel,
    missing_method = "impute", mice_args = list(seed = 42L)
  )
  nrow(result_imp@data)
}
#> [1] 6422

# --- GEE estimating equation ------------------------------------------
# GEE needs population-scale reference weights, or the solver diverges.
result_gee <- ipw(
  ns_complete, npors_ref, selection = sel, estimating_eq = "gee"
)
summarize_weights(result_gee)
#> # A tibble: 1 × 11
#>       n n_positive n_zero   mean    cv    min    p25    p50    p75     max   ess
#>   <int>      <int>  <int>  <dbl> <dbl>  <dbl>  <dbl>  <dbl>  <dbl>   <dbl> <dbl>
#> 1  6302       6302      0 39659. 0.953 10814. 19702. 26514. 42360. 434326. 3304.

# --- known population size --------------------------------------------
result_known_n <- ipw(
  ns_wave1, gss_ref, selection = ~sex + age_f3, population_size = 258000000L
)
result_known_n@metadata@weighting_history[[1]]$population_size_known
#> [1] TRUE
```
