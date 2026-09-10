# Pew 2016 ATP synthetic population

A 20,000-row synthetic population dataset derived from the 2016 American
Trends Panel (ATP) probability sample, used as the reference population
for the Mercer, Lau and Kennedy (2018) opt-in weighting study.
Population truth values for the 13 benchmark variables can be computed
as unweighted means of the dichotomized benchmark columns.

Value labels are preserved as named numeric attributes on each column
(accessible via `attr(pew_2016_synth_pop$gender, "labels")`). Variable
labels are stored in the `"label"` attribute of each column.

Binary benchmark variables (shared with
[pew_2016_optin](https://jdenn0514.github.io/surveywts/reference/pew_2016_optin.md))
are dichotomised to `1` = positive response, `0` = negative response; no
Refused codes exist in this dataset.

## Usage

``` r
pew_2016_synth_pop
```

## Format

A data frame with 20,000 rows and 43 columns:

- id:

  Numeric. Row identifier.

- gender:

  Numeric. `1` = Male, `2` = Female.

- age:

  Numeric (continuous). Age in years.

- racethn:

  Numeric. Race/ethnicity: `1` = White non-Hispanic, `2` = Black
  non-Hispanic, `3` = Hispanic, `4` = Asian, `5` = Other race. No
  Refused category.

- educcat5:

  Numeric. 5-category education: `1` = Less than HS, `2` = HS Grad, `3`
  = Some college, `4` = College grad, `5` = Postgraduate. No Refused
  category.

- division:

  Numeric. Census division, **alphabetical coding**: `1` = East North
  Central, `2` = East South Central, `3` = Middle Atlantic, `4` =
  Mountain, `5` = New England, `6` = Pacific, `7` = South Atlantic, `8`
  = West North Central, `9` = West South Central. Coding matches
  [pew_2016_optin](https://jdenn0514.github.io/surveywts/reference/pew_2016_optin.md).

- marital_acs:

  Numeric. Marital status (ACS-sourced).

- hhsizecat:

  Numeric. Household size category.

- childrencat:

  Numeric. Number of children category.

- citizen_rec:

  Numeric. U.S. citizenship.

- born_acs:

  Numeric. Born in the U.S.

- faminc5:

  Numeric. Family income (5 categories).

- employed:

  Numeric. Employment status (3 categories).

- worker_class:

  Numeric. Employment sector (class of worker).

- usual_hrs_per_week:

  Numeric. Hours worked per week.

- hours_vary:

  Numeric. Hours worked per week vary.

- mil_acs_rec:

  Numeric. Military status.

- home_acs_rec:

  Numeric. Home ownership.

- metropolitan:

  Numeric. Lives in a metropolitan statistical area.

- internet_access:

  Numeric. Household internet access.

- fdstmp_cps:

  Integer. Household received food stamps: `1` = Yes, `0` = No.

- tenure_acs:

  Numeric. Lived in home one year ago.

- pub_off_cps:

  Integer. Contacted public official in last 12 months: `1` = Yes, `0` =
  No.

- boycott:

  Numeric. Boycotted a product/service in last 12 months.

- comgrp_cps:

  Integer. Participated in community group in last 12 months: `1` = Yes,
  `0` = No.

- talk_cps:

  Numeric. Frequency of talking with neighbors (ordinal): `1` =
  Basically every day, `2` = A few times a week, `3` = A few times a
  month, `4` = Rarely, `5` = Not at all.

- trust_cps:

  Numeric. Trust in neighbors (ordinal): `1` = All of the people, `2` =
  Most, `3` = Some, `4` = None.

- tablet_cps:

  Integer. Household has a tablet or e-reader: `1` = Yes, `0` = No.

- textim_cps:

  Integer. Uses texting or instant messaging: `1` = Yes, `0` = No.

- social_cps:

  Integer. Uses social networking: `1` = Yes, `0` = No.

- volsum:

  Integer. Volunteered in last 12 months: `1` = Volunteered, `0` = Did
  not volunteer.

- registered:

  Integer. Registered to vote: `1` = Yes, `0` = No. Recoded from the raw
  SPSS file (original: `1` = No, `2` = Yes).

- vote14:

  Integer. Voted in 2014: `1` = Voted, `0` = Did not vote. Recoded from
  the raw SPSS file (original: `1` = Did not vote, `2` = Voted).

- partyscale5:

  Numeric. Party ID: `1` = Republican, `2` = Lean Republican, `3` =
  Ind/No Lean, `4` = Lean Democrat, `5` = Democrat.

- religcat:

  Numeric. Religion category (6 levels).

- ideo3:

  Numeric. Ideology: `1` = Liberal, `2` = Moderate, `3` = Conservative.

- folgov:

  Numeric. Follows government/public affairs (ordinal): `1` = Most of
  the time, `2` = Some of the time, `3` = Only now and then, `4` =
  Hardly at all.

- owngun_gss:

  Integer. Gun in home: `1` = Yes, `0` = No.

- sex:

  Factor. Derived from `gender`: `1` = `"Male"`, `2` = `"Female"`.
  Levels: `c("Male", "Female")`.

- race_f4:

  Factor. Derived from `racethn`: `1` = `"White"`, `2` = `"Black"`, `3`
  = `"Hispanic"`, `4`/`5` = `"Other"`. Levels:
  `c("White", "Black", "Hispanic", "Other")`.

- edu_f3:

  Factor. Derived from `educcat5`: `1` = `"Less than HS"`, `2:3` =
  `"HS/Some college"`, `4:5` = `"College+"`. Levels:
  `c("Less than HS", "HS/Some college", "College+")`.

- pid_f3:

  Factor. Derived from `partyscale5`: `1:2` = `"Republican"`, `3` =
  `"Independent"`, `4:5` = `"Democrat"`. Levels:
  `c("Republican", "Independent", "Democrat")`.

- age_f3:

  Factor. Derived from `age` using
  `cut(age, breaks = c(18, 35, 55, Inf), right = FALSE)`. Levels:
  `c("18-34", "35-54", "55+")`.

## Source

Derived from the Pew Research Center 2016 ATP synthetic population SPSS
file. See `data-raw/pew-2016-synth-pop.R` for the preparation script.

## Details

Pew 2016 ATP synthetic population

## See also

[pew_2016_optin](https://jdenn0514.github.io/surveywts/reference/pew_2016_optin.md)
