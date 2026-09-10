# Pew NPORS 2025 complete cases only

A filtered version of
[npors_2025](https://jdenn0514.github.io/surveywts/reference/npors_2025.md)
with rows removed where any of `sex`, `age_f3`, `race_f4`, `edu_f3`, or
`pid_f3` is `NA`. Approximately 4,814 rows are retained (5,022 minus
approximately 208 rows with at least one `NA` in the five derived
columns). Weights are not re-normalized after row removal.

NPORS is a national address-based probability sample with stratified
random sampling and differential probabilities of selection across
strata. Always include `strata = stratum` when constructing a survey
design from this data. To construct a survey design for standard
estimation:

    data(npors_2025_clean)
    npors_design <- surveycore::as_survey(
      npors_2025_clean, weights = weight, strata = stratum
    )

For IPW use, construct a reference design using `wt_pop`:

    ref <- surveycore::as_survey(
      npors_2025_clean, weights = wt_pop, strata = stratum
    )
    ipw(ns_wave1, ref,
        selection = ~sex + age_f3 + race_f4 + edu_f3,
        missing_method = "omit")

Use this object when passing to
[`ipw()`](https://jdenn0514.github.io/surveywts/reference/ipw.md) to
avoid the `surveywts_warning_ipw_reference_na_omitted` warning. Use
[npors_2025](https://jdenn0514.github.io/surveywts/reference/npors_2025.md)
when downstream code handles missingness itself.

## Usage

``` r
npors_2025_clean
```

## Format

### `npors_2025_clean`

A data frame with approximately 4,814 rows and 71 columns (same column
structure as
[npors_2025](https://jdenn0514.github.io/surveywts/reference/npors_2025.md)).
No `NA` values in `sex`, `age_f3`, `race_f4`, `edu_f3`, or `pid_f3`.

- respid:

  Character. Respondent identifier.

- mode:

  Numeric. Interview mode.

- language:

  Numeric. Interview language.

- languageinitial:

  Numeric. Language at initial contact.

- stratum:

  Numeric. Sampling stratum.

- interview_start:

  Character. Interview start date/time.

- interview_end:

  Character. Interview end date/time.

- econ1mod:

  Numeric. Economic conditions assessment.

- econ1bmod:

  Numeric. Economic conditions (follow-up).

- comtype2:

  Numeric. Community type.

- unity:

  Numeric. National unity opinion.

- crimesafe:

  Numeric. Crime and safety opinion.

- govprotct:

  Numeric. Government protection opinion.

- moregunimpact:

  Numeric. Opinion on gun impact.

- fin_sit:

  Numeric. Financial situation.

- vet1:

  Numeric. Veteran status.

- vol12_cps:

  Numeric. Volunteered in last 12 months.

- eminuse:

  Numeric. Email use.

- intmob:

  Numeric. Internet mobile use.

- intfreq:

  Numeric. Internet frequency.

- intfreq_collapsed:

  Numeric. Internet frequency (collapsed).

- home4nw2:

  Numeric. Home internet access.

- bbhome:

  Numeric. Broadband at home.

- smuse_fb:

  Numeric. Uses Facebook.

- smuse_yt:

  Numeric. Uses YouTube.

- smuse_x:

  Numeric. Uses X (formerly Twitter).

- smuse_ig:

  Numeric. Uses Instagram.

- smuse_sc:

  Numeric. Uses Snapchat.

- smuse_wa:

  Numeric. Uses WhatsApp.

- smuse_tt:

  Numeric. Uses TikTok.

- smuse_rd:

  Numeric. Uses Reddit.

- smuse_bsk:

  Numeric. Uses Bluesky.

- smuse_th:

  Numeric. Uses Threads.

- smuse_ts:

  Numeric. Uses Tumblr.

- radio:

  Numeric. Listens to radio.

- device1a:

  Numeric. Device ownership.

- smart2:

  Numeric. Smartphone ownership.

- nhisll:

  Numeric. Health insurance.

- relig:

  Numeric. Religion.

- religcat1:

  Numeric. Religion category.

- born:

  Numeric. Born-again Christian.

- attendper:

  Numeric. Religious attendance (in-person).

- attendonline2:

  Numeric. Religious attendance (online).

- relimp:

  Numeric. Importance of religion.

- pray:

  Numeric. Prayer frequency.

- educcat:

  Numeric. Education category (raw): `1` = College+, `2` = HS/Some
  college, `3` = Less than HS.

- registration:

  Numeric. Voter registration status.

- party:

  Numeric. Political party identification.

- partyln:

  Numeric. Party leaning (for independents).

- partysum:

  Numeric. Party summary.

- hisp:

  Numeric. Hispanic origin: `1` = Yes, `2` = No.

- racecmb:

  Numeric. Race (combined).

- racethn:

  Numeric. Race/ethnicity (raw).

- agegrp:

  Numeric. Age group (raw).

- agecat:

  Numeric. Age category (alternative grouping).

- birthplace:

  Numeric. Born in the United States.

- gender:

  Numeric. Interview gender: `1` = Male, `2` = Female, `3` = Non-binary,
  `99` = Refused.

- adults:

  Numeric. Number of adults in household.

- voted2024:

  Numeric. Voted in 2024.

- votegen_post:

  Numeric. 2024 vote choice.

- inc_sdt1:

  Numeric. Income.

- cregion:

  Numeric. Census region.

- metro:

  Numeric. Metropolitan area.

- basewt:

  Numeric. Base weight before calibration.

- weight:

  Numeric. Final normalized survey weight. Use for standard estimation.
  For IPW use `wt_pop` instead.

- sex:

  Factor. `"Male"` or `"Female"`. No `NA` values. Levels:
  `c("Male", "Female")`.

- age_f3:

  Factor. `"18-34"`, `"35-54"`, or `"55+"`. No `NA` values. Levels:
  `c("18-34", "35-54", "55+")`.

- race_f4:

  Factor. Race/ethnicity (4-level). No `NA` values. Levels:
  `c("White", "Black", "Hispanic", "Other")`.

- edu_f3:

  Factor. Education level. No `NA` values. Levels:
  `c("Less than HS", "HS/Some college", "College+")`.

- pid_f3:

  Factor. Party identification. No `NA` values. Levels:
  `c("Republican", "Independent", "Democrat")`.

- wt_pop:

  Numeric. Population-scaled weight: `weight * (260000000 / 5022)`.
  Weights are not re-normalized after row removal.

## Source

Derived from
[npors_2025](https://jdenn0514.github.io/surveywts/reference/npors_2025.md).
See `data-raw/npors-2025.R` for the construction script.

## See also

[npors_2025](https://jdenn0514.github.io/surveywts/reference/npors_2025.md),
[ns_wave1](https://jdenn0514.github.io/surveywts/reference/ns_wave1.md)
