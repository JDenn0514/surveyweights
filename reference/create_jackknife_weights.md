# Construct jackknife replicate weights

Generates jackknife replicate weights (sets of perturbed weight columns
used to compute standard errors) for probability samples via the
`survey` or `svrep` package, and for non-probability samples via the
DAGJK (delete-a-group jackknife) engine. The `type` argument selects the
jackknife variant; `survey_nonprob` inputs are restricted to
`type = "grouped"`.

## Usage

``` r
create_jackknife_weights(
  data,
  replicates = NULL,
  ...,
  type = c("jkn", "jk1", "grouped"),
  mse = TRUE,
  var_strat = NULL,
  var_strat_frac = NULL,
  sort_var = NULL,
  adj_method = c("variance-stratum-psus", "variance-units"),
  scale_method = c("variance-stratum-psus", "variance-units"),
  reference_sample = NULL,
  seed = NULL
)
```

## Arguments

- data:

  A `survey_taylor` or `survey_nonprob`. `survey_nonprob` is only valid
  with `type = "grouped"`. `data.frame` and `survey_replicate` inputs
  all error.

- replicates:

  `integer(1)` or `NULL`. Number of random deletion groups for
  `type = "grouped"`. Required when `type = "grouped"` for both
  `survey_taylor` and `survey_nonprob` inputs; errors if `NULL`.
  Silently ignored for `type = "jkn"` and `type = "jk1"` (those types
  are deterministic). For `survey_nonprob` (DAGJK path), must be a whole
  number (value \>= 2); whole-number doubles (e.g., `50.0`) are coerced
  to integer. Must not exceed the combined NPS + reference row count.
  Documentation advises `G >= 50` as a practical starting point for
  CV(SE) \<= 10% (Valliant, Dever & Kreuter 2018 Table 15.2). No default
  is provided by design — callers must always supply a value when
  `type = "grouped"`.

- ...:

  Must be empty. Forces all remaining arguments to be named.

- type:

  `character(1)`. `"jkn"` (the default): stratified delete-one
  jackknife, one replicate per PSU across all strata. `"jk1"`:
  unstratified delete-one jackknife ignoring stratification — will
  generally overestimate variance on multi-stratum designs. `"grouped"`:
  random-group jackknife for probability samples (via `svrep`) or DAGJK
  for non-probability samples. `"jkn"` and `"jk1"` are valid only for
  `survey_taylor`.

- mse:

  `logical(1)`, default `TRUE`. Centers each replicate deviation on the
  full-sample estimate (`mse = TRUE`, Wolter 2007 v_4 form;
  conservative) or on the within-stratum mean of replicate estimates
  (`mse = FALSE`, v_1 form). For `type = "grouped"` with
  `survey_nonprob` (DAGJK), `mse = TRUE` is hardcoded — see the
  **Algorithm** section. Supplying `mse = FALSE` on that path emits a
  warning and is overridden.

- var_strat:

  `character(1)` or `NULL`. Variance stratification variable; passed to
  [`svrep::as_random_group_jackknife_design()`](https://bschneidr.github.io/svrep/reference/as_random_group_jackknife_design.html)
  for `type = "grouped"` with `survey_taylor`. Silently ignored for
  `"jkn"` and `"jk1"`. Emits a warning and is ignored for
  `survey_nonprob` (DAGJK does not support svrep variance
  stratification).

- var_strat_frac:

  `numeric(1)` or `NULL`. Variance stratification fraction for `svrep`.
  Same applicability and ignored-path behavior as `var_strat`.

- sort_var:

  `character(1)` or `NULL`. Sort variable for systematic group
  assignment in `svrep`. Same applicability and ignored-path behavior as
  `var_strat`. Not applicable to DAGJK because DAGJK uses its own
  [`sample()`](https://rdrr.io/r/base/sample.html) engine across the
  combined NPS + reference dataset.

- adj_method:

  `character(1)`. `"variance-stratum-psus"` (the default, matching the
  svrep default) or `"variance-units"`. Passed to
  [`svrep::as_random_group_jackknife_design()`](https://bschneidr.github.io/svrep/reference/as_random_group_jackknife_design.html)
  for `type = "grouped"` with `survey_taylor`. Silently ignored for
  `"jkn"` and `"jk1"`. Emits a warning when non-default and `data` is
  `survey_nonprob`.

- scale_method:

  `character(1)`. `"variance-stratum-psus"` (the default) or
  `"variance-units"`. Same applicability and ignored-path behavior as
  `adj_method`.

- reference_sample:

  `survey_taylor` or `NULL`. Reference probability sample for the DAGJK
  path (`type = "grouped"` with `survey_nonprob`). When non-`NULL`,
  overrides any reference design stored in the
  [`ipw()`](https://jdenn0514.github.io/surveywts/reference/ipw.md)
  history entry. When non-`NULL` and not a `survey_taylor`, errors with
  `surveywts_error_reference_sample_class`. Silently ignored when `type`
  is `"jkn"`, `"jk1"`, or `"grouped"` with `survey_taylor` (documented
  as DAGJK-only; no runtime warning emitted on non-DAGJK paths).

- seed:

  `integer(1)` or `NULL`. RNG seed for reproducible random group
  assignment. For `type = "grouped"` with `survey_taylor`, passed to
  `.convert_and_call()` which calls
  [`withr::local_seed()`](https://withr.r-lib.org/reference/with_seed.html).
  For DAGJK, `set.seed(seed)` is called once before group assignment;
  the global RNG state is not restored. Silently ignored for `"jkn"` and
  `"jk1"` (deterministic).

## Value

**`type = "jkn"`, `type = "jk1"`, or `type = "grouped"` with
`survey_taylor`:** A `survey_replicate` with replicate weight columns in
`@data` named `rep_1`, `rep_2`, ..., `rep_R`; `@variables$repweights`
populated; `@variables$type` set to `"JKn"`, `"JK1"`, or
`"random-group"` respectively; `@variables$scale`, `@variables$rscales`,
and `@variables$mse` populated from the backend. A new entry with
`operation = "replicate_creation"` and `method = "jackknife"` is
appended to the weighting history.

**`type = "grouped"` with `survey_nonprob`:** A `survey_nonprob` with
`G_success` replicate weight columns named `repwt_1` through
`repwt_{G_success}`; `@variables$type` set to `"group-jackknife"`;
`@variables$scale` set to `(G_success - 1) / G_success`;
`@variables$rscales` set to `rep(1, G_success)`; `@variables$mse` set to
`TRUE`. A new entry with `operation = "jackknife_weights"` is appended
to the weighting history.

## Details

**When to use.** Choose the jackknife for a stratified, possibly
multi-stage probability sample where every stratum holds two or more
PSUs (primary sampling units: the first units the design selects, such
as counties or schools) and you estimate a total, a mean, or a ratio
(Valliant, Dever & Kreuter 2018, Section 15.4). Do not use it for a
median or another quantile — the jackknife standard error for a quantile
does not improve as the sample grows; use
[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md)
or, on a paired design,
[`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md).
When every stratum holds exactly two PSUs, prefer BRR.

## Algorithm

**JKn (stratified delete-one jackknife)**

Drops PSU `i` from stratum `h` in turn. Retained units in stratum `h`
are scaled by `n_h / (n_h - 1)`; units in other strata are unchanged:

\$\$ w\_{k(hi)} = \begin{cases} 0 & \text{if unit } k \text{ is in PSU }
i \text{ of stratum } h \\ \dfrac{n_h}{n_h - 1}\\ w_k & \text{if unit }
k \text{ is in stratum } h,\\ k \neq i \\ w_k & \text{if unit } k \text{
is not in stratum } h \end{cases} \$\$

The `mse = TRUE` variance estimator (Wolter 2007 eq. 4.6.4a):

\$\$ v_J(\hat{\theta}) = \sum\_{h=1}^{H} \frac{n_h - 1}{n_h}
\sum\_{i=1}^{n_h} \left( \hat{\theta}\_{(hi)} - \hat{\theta} \right)^2
\$\$

Total replicates = \\\sum_h n_h\\ (one per PSU across all strata).
Delegated to `survey::as.svrepdesign(type = "JKn", mse = mse)`.

**JK1 (unstratified delete-one jackknife)**

Special case of JKn treating all PSUs as a single stratum (Valliant,
Dever & Kreuter 2018 §15.4.1 "Special Cases"). Scale factor is
`(n-1)/n`; total replicates = `n`. JK1 ignores the design stratification
and generally overestimates variance when applied to a multi-stratum
design. Delegated to `survey::as.svrepdesign(type = "JK1", mse = mse)`.

**Grouped jackknife (probability samples)**

PSUs are randomly divided into `replicates` groups. Each replicate drops
one group and rescales the remaining units' weights. Scale factor =
`(G - 1) / G` for equal-sized groups. Delegated to
[`svrep::as_random_group_jackknife_design()`](https://bschneidr.github.io/svrep/reference/as_random_group_jackknife_design.html)
which handles unequal groups and variance stratification via
`var_strat`, `var_strat_frac`, `sort_var`, `adj_method`, and
`scale_method`.

**Delete-a-group jackknife for non-probability samples (DAGJK)**

Randomly partitions the combined NPS + reference dataset into
`replicates` groups, then refits the full estimation pipeline (IPW
and/or calibration) on the leave-one-group-out subsample for each
replicate. The variance estimator (Kott 2001 eq. 1; Valliant 2020 eq.
3):

\$\$ v_J(\hat{\theta}) = \frac{G - 1}{G} \sum\_{g=1}^{G} \left(
\hat{\theta}\_{(g)} - \hat{\theta} \right)^2 \$\$

Centering is always on the full-sample estimate \\\hat{\theta}\\
(`mse = TRUE`); `mse = FALSE` is not valid for this formula.

The standard per-stratum weight adjustment (Kott 2001 §1):

\$\$ w\_{k(g)} = \begin{cases} 0 & \text{if PSU } j \text{ is in group }
g \\ \dfrac{n_h}{n_h - n\_{hg}}\\ w_k & \text{otherwise} \end{cases}
\$\$

When any NPS stratum has \\n_h \< G\\, the extended formula (Kott 2001
§3 eq. 2) is applied per stratum:

\$\$ Z = \sqrt{\frac{G}{(G-1)\\ n_h\\ (n_h - 1)}} \$\$ \$\$
w\_{k(g)}^{(E)} = \begin{cases} w_k & \text{if no PSU from stratum } h
\text{ is in group } g \\ w_k \bigl(1 - (n_h - 1)\\ Z\bigr) & \text{if
PSU containing } k \text{ is in group } g \\ w_k (1 + Z) & \text{if PSU
is in stratum } h \text{ but not in group } g \end{cases} \$\$

When \\n_h = 2\\, the deleted-PSU multiplier \\1 - Z = 1 -
\sqrt{G/(G-1)} \< 0\\ for all finite \\G\\; the resulting negative
replicate weights are retained and reported via a warning (see
**Warnings**).

## Limitations

**Jackknife is not consistent for quantile variance.** The jackknife
underestimates variance for quantiles and other non-smooth statistics
(Elliott & Valliant 2017 §4.1; Valliant, Dever & Kreuter 2018 §15.4.1;
Wolter 2007 §4.2.4). Use
[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md)
for quantile variance.

**No formal consistency proof for DAGJK on non-probability samples.**
Consistency is claimed by analogy to Krewski & Rao (1981); no proof
exists for the non-probability sample case (Valliant 2020 §2.4).
Variance estimates are asymptotically justified approximations.

**DAGJK captures only sample-based estimation variance.** The nonsample
variance component (uncertainty about unobserved population units) is
not captured. No finite-population correction is applied.

**DAGJK standard errors are slightly conservative for doubly-robust
estimators.** When the pipeline applies both IPW and downstream
calibration, DAGJK standard error estimates are positively biased by
approximately 3-5% at n = 500 (Valliant 2020 Table 8). This bias does
not vanish as n grows.

**JK1 ignores stratification.** Applying `type = "jk1"` to a
multi-stratum design treats all PSUs as coming from one stratum and
generally overestimates variance. Use `type = "jkn"` for stratified
designs.

## Warnings

If existing replicate weight columns are detected on a `survey_nonprob`
input (DAGJK path), they are cleared and a warning is emitted before
proceeding.

If the average DAGJK group size is fewer than 5 units, a warning is
emitted. Very small groups cause the propensity model to fail in some
replicates. Reduce `replicates` or use a larger combined NPS + reference
dataset.

If one or more DAGJK replicate weight values are negative after
calibration (which can occur with the extended formula when \\n_h = 2\\,
or with extreme calibration targets), a warning is emitted. Variance
estimates using negative replicate weights should be interpreted
cautiously.

## Messages

When the design carries a calibration, the replay re-runs it inside
every replicate. Replicates that already meet their margins are counted
and reported in one summary line, not announced one by one. A count
close to `replicates` means the replay changed little, so the variance
estimate may be near zero and deserves a look.

The replicate weight creators re-emit the svrep back end's notices under
their own classes, so you can quiet one without quieting the rest. Which
message you see depends on the function and the arguments.

- `surveywts_message_row_order_assumed` — from
  [`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md)
  and
  [`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
  when `variance_estimator` is `"SD1"` or `"SD2"`. Both estimators read
  the row order of the data, and nothing here checks that order.

- `surveywts_message_replicates_rounded_up` — from
  [`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md),
  when the Hadamard matrix order is above `replicates`. The result then
  carries more replicate columns than you asked for, and
  `use_normal_hadamard` selects which orders are reachable: at the
  default, `replicates = 100` gives 128.

- `surveywts_message_replicates_subsampled` — from
  [`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
  when `max_replicates` is below the fully efficient replicate count.
  The back end keeps a random sample of the replicates, so set `seed` to
  make the draw reproducible.

- `surveywts_message_backend_note` — from any of them, when the back end
  emits a message this package does not recognise. The text is
  unchanged.

To quiet one message and leave the rest alone:

    withCallingHandlers(
      create_sdr_weights(design, replicates = 100L),
      surveywts_message_replicates_rounded_up = function(cnd) {
        invokeRestart("muffleMessage")
      }
    )

## References

Elliott, M.R. and Valliant, R. (2017). Inference for nonprobability
samples. *Statistical Science* **32**(2), 249–264.
[doi:10.1214/16-STS598](https://doi.org/10.1214/16-STS598)

Kott, P.S. (2001). The delete-a-group jackknife. *Journal of Official
Statistics* **17**(4), 521–526.

Valliant, R. (2020). Comparing alternatives for estimation from
nonprobability samples. *Journal of Survey Statistics and Methodology*
**8**, 231–263.
[doi:10.1093/jssam/smz003](https://doi.org/10.1093/jssam/smz003)

Valliant, R., Brick, J.M. and Dever, J.A. (2008). Weight adjustments for
the grouped jackknife variance estimator. *Journal of Official
Statistics* **24**(3), 469–488.

Valliant, R., Dever, J.A. and Kreuter, F. (2018). *Practical Tools for
Designing and Weighting Survey Samples* (2nd ed.). Springer.

Wolter, K.M. (2007). *Introduction to Variance Estimation* (2nd ed.).
Springer.

## See also

[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md)
for bootstrap replicate weights;
[`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md)
for balanced repeated replication;
[`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md)
for generalized bootstrap;
[`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md)
for generalized replication;
[`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md)
for successive difference replication;
[`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md)
for the method-dispatch wrapper;
[`as_taylor_design()`](https://jdenn0514.github.io/surveywts/reference/as_taylor_design.md)
to recover the Taylor-linearization design from replicate-weight
history.

For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other replicate-weights:
[`as_taylor_design()`](https://jdenn0514.github.io/surveywts/reference/as_taylor_design.md),
[`create_bootstrap_weights()`](https://jdenn0514.github.io/surveywts/reference/create_bootstrap_weights.md),
[`create_brr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_brr_weights.md),
[`create_gen_boot_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_boot_weights.md),
[`create_gen_rep_weights()`](https://jdenn0514.github.io/surveywts/reference/create_gen_rep_weights.md),
[`create_replicate_weights()`](https://jdenn0514.github.io/surveywts/reference/create_replicate_weights.md),
[`create_sdr_weights()`](https://jdenn0514.github.io/surveywts/reference/create_sdr_weights.md)

## Examples

``` r
# JKn (stratified delete-one) on a probability sample --------------------
gss_svy <- surveycore::as_survey(
  gss_2024, weights = wtssps, strata = vstrat, ids = vpsu, nest = TRUE
)
jkn_design <- create_jackknife_weights(gss_svy, type = "jkn")
summarize_weights(jkn_design)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  3309       3309      0     1 0.979 0.151 0.384 0.717  1.15  8.76 1689.
# The confidence interval below is computed from the delete-one
# replicate columns rather than by Taylor linearization.
surveycore::get_means(jkn_design, age)
#> # A tibble: 1 × 4
#>    mean ci_low ci_high     n
#>   <dbl>  <dbl>   <dbl> <int>
#> 1  47.9   47.0    48.9  3208

# Grouped jackknife on a probability sample -------------------------------
# Every one of `gss_2024`'s 67 strata holds exactly 2 PSUs, and the grouped
# jackknife cannot ask for more replicates than the smallest stratum has
# PSUs. 2 is the maximum this design allows, not a low setting: 3L errors.
grouped_design <- create_jackknife_weights(
  gss_svy,
  replicates = 2L,
  type = "grouped",
  seed = 42L
)
summarize_weights(grouped_design)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  3309       3309      0     1 0.979 0.151 0.384 0.717  1.15  8.76 1689.

# DAGJK on a calibrated non-probability sample ----------------------------
targets_a <- list(
  sex    = c("Male" = 0.49, "Female" = 0.51),
  age_f3 = c("18-34" = 0.30, "35-54" = 0.33, "55+" = 0.37)
)
ns_wave1_svy <- surveycore::as_survey_nonprob(ns_wave1, weights = weight)
ns_wave1_cal <- calibrate_rake(ns_wave1_svy, targets = targets_a)
# This path replays the calibration inside every replicate. Replicates that
# already met their margins are named in one summary line. Real analyses use
# more replicates; 25 keeps `R CMD check` fast.
dagjk_design <- create_jackknife_weights(
  ns_wave1_cal,
  replicates = 25L,
  type = "grouped",
  seed = 42L
)
#> ℹ Raking converged in 1 sweep in 22 of 25 replicates: those replicates already
#>   met their margins.
summarize_weights(dagjk_design)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv     min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  6422       6422      0     1  1.36 0.00373 0.152 0.399  1.13  4.96 2248.
```
