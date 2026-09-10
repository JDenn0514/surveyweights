# Correct weights for unit nonresponse

Redistributes the weights of nonrespondents to respondents within
weighting classes (groups assumed to share one response rate) defined by
`by`. Respondent weights increase proportionally to preserve the total
weight within each class. What happens to the nonrespondent rows depends
on the input class — see the **Input class behavior** section.

## Usage

``` r
adjust_nonresponse(
  data,
  response_status,
  weights = NULL,
  by = NULL,
  wt_name = NULL,
  method = c("weighting-class", "propensity-cell", "propensity"),
  formula = NULL,
  control = list(min_cell = 20, max_adjust = 2, n_cells = 5)
)
```

## Arguments

- data:

  A `survey_taylor` or `survey_nonprob`. Must include BOTH respondents
  and nonrespondents. `survey_replicate` -\> error. Any other class -\>
  error.

- response_status:

  Bare name (NSE). Binary response indicator column. Must be `logical`
  or integer `0`/`1`. `1` / `TRUE` = respondent.

- weights:

  Bare name (NSE). Weight column. `NULL` -\> auto-detected from survey
  object `@variables$weights`.

- by:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Weighting class variables. Redistribution is performed within each
  cell defined by the joint combination of these variables. `NULL` -\>
  global redistribution across all rows.

- wt_name:

  `NULL` (default) or a character scalar. When `NULL`, adjusted weights
  overwrite the existing weight column in place. When a character
  string, a new column is added and `@variables$weights` updated.

- method:

  Character scalar. Adjustment method. One of `"weighting-class"`
  (default), `"propensity-cell"`, or `"propensity"`.

  - `"weighting-class"`: cells are defined by `by` groups; adjustment
    factors are applied cell-by-cell. Choose it when you can name the
    groups whose response rates differ, and those groups are observed
    for respondents and nonrespondents alike.

  - `"propensity-cell"`: fits a logistic response propensity model via
    `formula`, bins scores into `control$n_cells` quantile cells, then
    applies cell-level adjustments. Choose it when no natural grouping
    exists: the model builds the cells from the data, and the cell-level
    factor smooths over individual fitted propensities.

  - `"propensity"`: fits a logistic response propensity model via
    `formula` and applies individual-level inverse-probability weights
    (`weight_i / propensity_i`) to each respondent. Requires `formula`.
    Choose it when each unit's own fitted propensity should set its
    adjustment; individual factors can move further than cell-level
    ones, so watch the `max_adjust` warning.

- formula:

  A one-sided formula (e.g., `~ age_group + sex`) used for propensity
  score estimation when `method = "propensity-cell"` or
  `method = "propensity"`. Required for `"propensity"`. All variables
  must be present in `data` with no `NA` values.

- control:

  Named list of control parameters. Merged with defaults
  `list(min_cell = 20, max_adjust = 2.0, n_cells = 5)`.

  - `min_cell`: warn when a cell has fewer than this many respondents
    (default 20, per NAEP methodology). Used only for
    `"propensity-cell"`.

  - `max_adjust`: warn when the nonresponse adjustment factor exceeds
    this value (default 2.0). For `"propensity-cell"`, the cell-level
    factor; for `"propensity"`, the individual IPW ratio relative to the
    mean respondent weight.

  - `n_cells`: number of propensity score cells (default 5). Must be a
    whole number \>= 2. Used only when `method = "propensity-cell"`.
    Either `min_cell` or `max_adjust` condition alone triggers the
    warning.

## Value

An object of the same class as `data` with adjusted weights. The rows it
contains depend on the input class (see the **Input class behavior**
section):

- `survey_nonprob` input -\> `survey_nonprob` with all rows;
  nonrespondent weights are set to 0

- `survey_taylor` input -\> `survey_taylor` with respondent rows only,
  because `survey_taylor` does not support zero weights

A history entry with `operation = "nonresponse_weighting_class"` (for
`method = "weighting-class"`),
`operation = "nonresponse_propensity_cell"` (for
`method = "propensity-cell"`), or `operation = "nonresponse_propensity"`
(for `method = "propensity"`) is appended to `weighting_history`.

## Details

**When to use.** This function corrects for known nonrespondents inside
your own sample; it needs a column that marks who responded. To move
weight between rows defined by other conditions, use
[`redistribute_weights()`](https://jdenn0514.github.io/surveywts/reference/redistribute_weights.md).
To weight a sample that has no response indicator against a reference
survey, use
[`ipw()`](https://jdenn0514.github.io/surveywts/reference/ipw.md).

For `survey_nonprob` input, zero-weight observations are retained for
design-based variance estimation. Survey estimation functions (e.g.,
[`survey::svymean()`](https://rdrr.io/pkg/survey/man/surveysummary.html))
handle zero weights correctly – zero-weight units are excluded from
point estimates but included in the design structure for variance
estimation. For manual calculations, use `w[w > 0]` to exclude
nonrespondents. For `survey_taylor` input, the returned object carries
no zero-weight rows – the nonrespondent rows are dropped.

Diagnostic functions
([`effective_sample_size()`](https://jdenn0514.github.io/surveywts/reference/effective_sample_size.md),
[`weight_variability()`](https://jdenn0514.github.io/surveywts/reference/weight_variability.md),
[`summarize_weights()`](https://jdenn0514.github.io/surveywts/reference/summarize_weights.md))
automatically filter to positive weights before computing statistics.

Re-calibrating post-nonresponse data requires filtering to respondents
first, because
[`calibrate()`](https://jdenn0514.github.io/surveywts/reference/calibrate.md),
[`calibrate_rake()`](https://jdenn0514.github.io/surveywts/reference/calibrate_rake.md),
and
[`poststratify()`](https://jdenn0514.github.io/surveywts/reference/poststratify.md)
reject zero weights.

**Propensity-cell method:** A logistic regression is fitted via
[`stats::glm()`](https://rdrr.io/r/stats/glm.html) with
`family = quasibinomial`. The quasibinomial family gives the same
coefficients as `binomial` and does not warn about non-integer counts,
which survey weights always produce. GLM convergence warnings (e.g.,
fitted probabilities numerically 0 or 1) pass through unchanged. Cell
boundaries are defined by unweighted quantiles of the predicted
propensity scores (the modeled probability of responding or of appearing
in the sample). This method assumes missing at random (response depends
only on observed variables). The `by` argument is silently ignored (a
warning is issued).

## Note

The propensity-cell method assumes **Missing At Random (MAR)**: response
propensity is fully captured by the formula variables. Violations of MAR
(i.e., response depends on unobserved variables) cannot be detected or
corrected by this method. Additionally, propensity scores are treated as
known (estimated from the data), not as true population propensities;
this understates variance and should be accounted for in downstream
analysis.

## Input class behavior

The rows in the returned object depend on the class of `data`:

- `survey_nonprob`: all rows are returned. Nonrespondent rows are kept
  with a weight of zero.

- `survey_taylor`: respondent rows only are returned. The
  `survey_taylor` validator requires strictly positive weights, so the
  zero-weight nonrespondent rows are dropped.

In both cases respondent weights are adjusted upward to conserve the
total weight within each cell.

## Algorithm

Within each cell \\h\\ defined by `by`, the adjustment factor is \$\$f_h
= \frac{\sum\_{i \in h} w_i}{\sum\_{i \in h, \text{resp}} w_i}\$\$ where
the numerator sums all weights in the cell and the denominator sums
respondent weights only. Each respondent weight becomes \\w\_{i,new} =
w_i \times f_h\\. Nonrespondent weights are set to 0.

## See also

[`redistribute_weights()`](https://jdenn0514.github.io/surveywts/reference/redistribute_weights.md),
[`ipw()`](https://jdenn0514.github.io/surveywts/reference/ipw.md). For
the class system, the standard workflows, and a glossary of terms, see
the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other nonresponse:
[`redistribute_weights()`](https://jdenn0514.github.io/surveywts/reference/redistribute_weights.md)

## Examples

``` r
# survey_taylor: mutate the tibble first, then construct the design -------
# `formula` below needs no NA in sex or age_f3, so filter both here.
gss_with_resp <- gss_2024[!is.na(gss_2024$sex) & !is.na(gss_2024$age_f3), ]
gss_with_resp$responded <- sample(
  c(0L, 1L), nrow(gss_with_resp), replace = TRUE, prob = c(0.2, 0.8)
)
gss_svy <- surveycore::as_survey(
  gss_with_resp, weights = wtssps, strata = vstrat, ids = vpsu, nest = TRUE
)

# method = "weighting-class" (the default) ---------------------------------
result <- adjust_nonresponse(gss_svy, response_status = responded, by = sex)
summarize_weights(result)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  2571       2571      0  1.24 0.998 0.185 0.474 0.883  1.42  11.0 1288.

# survey_taylor requires strictly positive weights, so the zero-weight
# nonrespondent rows are dropped. The exact drop count moves with the
# unseeded draw above; the two counts below still differ every run.
nrow(surveycore::survey_data(gss_svy))
#> [1] 3197
nrow(surveycore::survey_data(result))
#> [1] 2571

# method = "propensity-cell": one factor per propensity cell ---------------
result_pc <- adjust_nonresponse(
  gss_svy,
  response_status = responded,
  method = "propensity-cell",
  formula = ~ sex + age_f3
)
summarize_weights(result_pc)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  2571       2571      0  1.24  1.00 0.182 0.472 0.878  1.42  11.2 1281.

# method = "propensity": one factor per unit ------------------------------
# Individual factors move further than cell-level ones, so this trips the
# `max_adjust` warning. The factor itself moves with the unseeded draw.
result_p <- adjust_nonresponse(
  gss_svy,
  response_status = responded,
  method = "propensity",
  formula = ~ sex + age_f3
)
#> Warning: ! The maximum propensity adjustment factor (11.42×) exceeds
#>   `control$max_adjust` (2×).
#> ℹ Large adjustment factors indicate strong nonresponse bias; weights may be
#>   highly variable.
#> ℹ Consider simplifying `formula`, using `trim_weights()`, or increasing
#>   `control$max_adjust` to suppress this warning.
summarize_weights(result_p)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  2571       2571      0  1.24  1.00 0.181 0.470 0.877  1.42  11.4 1280.
```
