# Transfer weight from excluded rows to retained rows

Removes the rows satisfying `reduce_if` and proportionally redistributes
their weight to rows satisfying `increase_if` within groups defined by
`by`. Rows matching neither condition keep their weight unchanged.

## Usage

``` r
redistribute_weights(
  data,
  reduce_if,
  increase_if,
  weights = NULL,
  by = NULL,
  wt_name = NULL,
  control = list()
)
```

## Arguments

- data:

  A `survey_taylor` or `survey_nonprob`. `survey_replicate` -\> error.
  Any other class -\> error.

- reduce_if:

  Bare name (NSE). Binary indicator column (`logical` or integer
  `0`/`1`). Rows where this is `TRUE`/`1` have their weight set to 0 and
  their weight redistributed.

- increase_if:

  Bare name (NSE). Binary indicator column. Rows where this is
  `TRUE`/`1` receive the redistributed weight.

- weights:

  Bare name (NSE). Weight column. Auto-detected from
  `@variables$weights`.

- by:

  \<[`tidy-select`](https://tidyselect.r-lib.org/reference/language.html)\>
  Grouping variable(s). Redistribution is performed within each group.
  `NULL` -\> global redistribution.

- wt_name:

  `NULL` (default) or a character scalar. When `NULL`, adjusted weights
  overwrite the existing weight column in place. When a character
  string, a new column is added and `@variables$weights` updated.

- control:

  Named list of warning thresholds. Merged with defaults
  `list(min_cell = 20, max_adjust = 2.0)`. `min_cell`: warn when a group
  has fewer than this many `increase_if` rows. `max_adjust`: warn when
  the adjustment factor exceeds this value.

## Value

- `survey_nonprob` or `survey_taylor` input -\> same class as input,
  with `reduce_if` rows removed (zero weights violate survey
  validators).

A history entry with `operation = "redistribute_weights"` is appended.

## Details

This function is the general form of [adjust_nonresponse(method =
"weighting-class")](https://jdenn0514.github.io/surveywts/reference/adjust_nonresponse.md).
When `reduce_if = nonrespondent indicator` and
`increase_if = respondent indicator`, the two produce equivalent results
(within `1e-10`). Reach for `redistribute_weights()` when the rows
losing weight are defined by a condition other than nonresponse — for
example, removing ineligible cases while conserving the group totals.

## Algorithm

Within each group \\h\\ defined by `by`, the adjustment factor applied
to each `increase_if` row is \$\$f_h = \frac{W\_{h,\text{reduce}} +
W\_{h,\text{increase}}}{W\_{h,\text{increase}}}\$\$ where
\\W\_{h,\text{reduce}}\\ and \\W\_{h,\text{increase}}\\ are the summed
weights of `reduce_if` and `increase_if` rows in group \\h\\. Each
`increase_if` weight becomes \\w\_{i,new} = w_i \times f_h\\. Rows
matching neither indicator are unchanged.

## See also

[`adjust_nonresponse()`](https://jdenn0514.github.io/surveywts/reference/adjust_nonresponse.md).
For the class system, the standard workflows, and a glossary of terms,
see the [Getting started
article](https://jdenn0514.github.io/surveywts/articles/getting-started.html).

Other nonresponse:
[`adjust_nonresponse()`](https://jdenn0514.github.io/surveywts/reference/adjust_nonresponse.md)

## Examples

``` r
# survey_taylor: mutate the tibble first, then construct the design -------
gss_excl <- gss_2024[!is.na(gss_2024$sex), ]
gss_excl$excluded <- sample(
  c(0L, 1L), nrow(gss_excl), replace = TRUE, prob = c(0.8, 0.2)
)
gss_excl$retained <- as.integer(!gss_excl$excluded)
gss_svy <- surveycore::as_survey(
  gss_excl, weights = wtssps, strata = vstrat, ids = vpsu, nest = TRUE
)
result <- redistribute_weights(
  gss_svy, reduce_if = excluded, increase_if = retained, by = sex
)
summarize_weights(result)
#> # A tibble: 1 × 11
#>       n n_positive n_zero  mean    cv   min   p25   p50   p75   max   ess
#>   <int>      <int>  <int> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#> 1  2566       2566      0  1.28 0.978 0.202 0.496 0.929  1.45  10.8 1311.
```
