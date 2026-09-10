# National Survey Wave 1 with harmonized demographic columns

The National Survey Wave 1 dataset, obtained from
[`surveycore::ns_wave1`](https://jdenn0514.github.io/surveycore/reference/ns_wave1.html).
All 6,422 respondents and 171 original columns are retained. The
`gender` column is kept as numeric. Five general-purpose derived columns
are added: `sex` (factor from `gender`), `age_f3` (factor from `age`),
`race_f4` (factor from `race_ethnicity` and `hispanic`), `edu_f3`
(factor from `education`), and `pid_f3` (factor from `pid3`). Eight
Nationscape raking recode columns (`ns_*`) are also added as calibration
variables used to replicate Nationscape's weighting procedure. Original
source columns are kept unchanged.

The `weight` column contains the raked+trimmed survey weight that
replicates Nationscape's published weighting procedure (Newton-Raphson,
10 marginal dimensions, 5th/95th percentile trim; Pearson r = 0.996 vs.
published weights). To construct a `survey_nonprob` for analysis:

    data(ns_wave1)
    ns_design <- surveycore::as_survey_nonprob(
      ns_wave1, weights = "weight"
    )

Some respondents have `NA` in `race_f4` (those who reported "some other
race" with no Hispanic origin, which cannot be mapped to a standard
category).

## Usage

``` r
ns_wave1
```

## Format

### `ns_wave1`

A data frame with 6,422 rows and 185 columns. All 171 original columns
from
[`surveycore::ns_wave1`](https://jdenn0514.github.io/surveycore/reference/ns_wave1.html)
are retained, including `gender` as numeric; fourteen new columns are
appended: six general-purpose derived columns (`sex`, `age_f3`,
`race_f4`, `edu_f3`, `pid_f3`, `hh_income_f9`) and eight Nationscape
raking recode columns (`ns_region`, `ns_hispanic`, `ns_race`, `ns_age`,
`ns_language`, `ns_foreign_born`, `ns_income`, `ns_vote_2016`):

- response_id:

  Character. Respondent identifier.

- start_date:

  Character. Survey start date.

- right_track:

  Numeric. Right track / wrong track opinion.

- economy_better:

  Numeric. Economy better/worse opinion.

- interest:

  Numeric. Political interest.

- registration:

  Numeric. Voter registration status.

- news_sources_facebook:

  Numeric. Gets news from Facebook.

- news_sources_cnn:

  Numeric. Gets news from CNN.

- news_sources_msnbc:

  Numeric. Gets news from MSNBC.

- news_sources_fox:

  Numeric. Gets news from Fox News.

- news_sources_network:

  Numeric. Gets news from network TV.

- news_sources_localtv:

  Numeric. Gets news from local TV.

- news_sources_telemundo:

  Numeric. Gets news from Telemundo.

- news_sources_npr:

  Numeric. Gets news from NPR.

- news_sources_amtalk:

  Numeric. Gets news from AM talk radio.

- news_sources_new_york_times:

  Numeric. Gets news from New York Times.

- news_sources_local_newspaper:

  Numeric. Gets news from local newspaper.

- news_sources_other:

  Numeric. Gets news from other sources.

- news_sources_other_TEXT:

  Character. Other news source (open text).

- pres_approval:

  Numeric. Presidential approval.

- vote_intention:

  Numeric. 2020 vote intention.

- vote_2016:

  Numeric. 2016 presidential vote.

- vote_2016_other_text:

  Character. 2016 vote other (open text).

- consider_trump:

  Numeric. Considers voting for Trump.

- not_trump:

  Numeric. Would not vote for Trump.

- primary_party:

  Numeric. Primary party preference.

- group_favorability_whites:

  Numeric. Favorability toward Whites.

- group_favorability_blacks:

  Numeric. Favorability toward Blacks.

- group_favorability_latinos:

  Numeric. Favorability toward Latinos.

- group_favorability_asians:

  Numeric. Favorability toward Asians.

- group_favorability_christians:

  Numeric. Favorability toward Christians.

- group_favorability_socialists:

  Numeric. Favorability toward Socialists.

- group_favorability_muslims:

  Numeric. Favorability toward Muslims.

- group_favorability_labor_unions:

  Numeric. Favorability toward labor unions.

- group_favorability_the_police:

  Numeric. Favorability toward the police.

- group_favorability_undocumented:

  Numeric. Favorability toward undocumented immigrants.

- group_favorability_lgbt:

  Numeric. Favorability toward LGBT people.

- group_favorability_republicans:

  Numeric. Favorability toward Republicans.

- group_favorability_democrats:

  Numeric. Favorability toward Democrats.

- cand_favorability_trump:

  Numeric. Favorability toward Trump.

- cand_favorability_obama:

  Numeric. Favorability toward Obama.

- cand_favorability_cortez:

  Numeric. Favorability toward Cortez.

- cand_favorability_biden:

  Numeric. Favorability toward Biden.

- cand_favorability_harris:

  Numeric. Favorability toward Harris.

- cand_favorability_buttigieg:

  Numeric. Favorability toward Buttigieg.

- cand_favorability_warren:

  Numeric. Favorability toward Warren.

- cand_favorability_sanders:

  Numeric. Favorability toward Sanders.

- cand_favorability_pence:

  Numeric. Favorability toward Pence.

- dem_vote_intent:

  Numeric. Democratic primary vote intent.

- dem_vote_intent_TEXT:

  Character. Democratic primary vote intent (open text).

- rank_dems_1:

  Numeric. Ranked choice: 1st choice Democrat.

- rank_dems_2:

  Numeric. Ranked choice: 2nd choice Democrat.

- rank_dems_3:

  Numeric. Ranked choice: 3rd choice Democrat.

- replace_trump:

  Numeric. Wants to replace Trump.

- house_intent:

  Numeric. House vote intent.

- senate_intent:

  Numeric. Senate vote intent.

- governor_intent:

  Numeric. Governor vote intent.

- trump_biden:

  Numeric. Trump vs. Biden preference.

- trump_sanders:

  Numeric. Trump vs. Sanders preference.

- trump_harris:

  Numeric. Trump vs. Harris preference.

- trump_warren:

  Numeric. Trump vs. Warren preference.

- trump_buttigieg:

  Numeric. Trump vs. Buttigieg preference.

- trump_booker:

  Numeric. Trump vs. Booker preference.

- trump_castro:

  Numeric. Trump vs. Castro preference.

- trump_gabbard:

  Numeric. Trump vs. Gabbard preference.

- trump_gillibrand:

  Numeric. Trump vs. Gillibrand preference.

- trump_orourke:

  Numeric. Trump vs. O'Rourke preference.

- pence_biden:

  Numeric. Pence vs. Biden preference.

- pence_buttigieg:

  Numeric. Pence vs. Buttigieg preference.

- pence_harris:

  Numeric. Pence vs. Harris preference.

- pence_sanders:

  Numeric. Pence vs. Sanders preference.

- pence_warren:

  Numeric. Pence vs. Warren preference.

- cand_truth_donald_trump:

  Numeric. Candidate truth rating: Trump.

- cand_truth_elizabeth_warren:

  Numeric. Candidate truth rating: Warren.

- cand_truth_joe_biden:

  Numeric. Candidate truth rating: Biden.

- cand_truth_bernie_sanders:

  Numeric. Candidate truth rating: Sanders.

- cand_truth_pete_buttigieg:

  Numeric. Candidate truth rating: Buttigieg.

- cand_truth_kamala_harris:

  Numeric. Candidate truth rating: Harris.

- cand_facts_donald_trump:

  Numeric. Candidate fact rating: Trump.

- cand_facts_elizabeth_warren:

  Numeric. Candidate fact rating: Warren.

- cand_facts_joe_biden:

  Numeric. Candidate fact rating: Biden.

- cand_facts_bernie_sanders:

  Numeric. Candidate fact rating: Sanders.

- cand_facts_pete_buttigieg:

  Numeric. Candidate fact rating: Buttigieg.

- cand_facts_kamala_harris:

  Numeric. Candidate fact rating: Harris.

- racial_attitudes_tryhard:

  Numeric. Racial attitude: trying too hard.

- racial_attitudes_generations:

  Numeric. Racial attitude: generations.

- racial_attitudes_marry:

  Numeric. Racial attitude: intermarriage.

- racial_attitudes_date:

  Numeric. Racial attitude: dating.

- gender_attitudes_maleboss:

  Numeric. Gender attitude: male boss.

- gender_attitudes_logical:

  Numeric. Gender attitude: logical.

- gender_attitudes_opportunity:

  Numeric. Gender attitude: opportunity.

- gender_attitudes_complain:

  Numeric. Gender attitude: complain.

- discrimination_blacks:

  Numeric. Perceived discrimination: Blacks.

- discrimination_whites:

  Numeric. Perceived discrimination: Whites.

- discrimination_muslims:

  Numeric. Perceived discrimination: Muslims.

- discrimination_christians:

  Numeric. Perceived discrimination: Christians.

- discrimination_women:

  Numeric. Perceived discrimination: women.

- discrimination_men:

  Numeric. Perceived discrimination: men.

- sen_knowledge:

  Numeric. Senate knowledge question.

- sc_knowledge:

  Numeric. Supreme Court knowledge question.

- pid3:

  Numeric. 3-category party identification.

- pid7_legacy:

  Numeric. 7-point party identification (legacy version).

- strength_democrat:

  Numeric. Strength of Democratic identification.

- strength_republican:

  Numeric. Strength of Republican identification.

- lean_independent:

  Numeric. Independent party leaning.

- ideo5:

  Numeric. Ideology (5-point scale).

- employment:

  Numeric. Employment status.

- employment_other_text:

  Character. Employment status (open text).

- foreign_born:

  Numeric. Born outside the United States.

- language:

  Numeric. Primary language.

- religion:

  Numeric. Religion.

- religion_other_text:

  Character. Religion (open text).

- is_evangelical:

  Numeric. Is evangelical or born-again Christian.

- orientation_group:

  Numeric. Sexual orientation.

- in_union:

  Numeric. Member of a labor union.

- household_gun_owner:

  Numeric. Household gun ownership.

- wall:

  Numeric. Opinion on border wall.

- cap_carbon:

  Numeric. Opinion on carbon cap.

- environment:

  Numeric. Environmental opinion.

- guns_bg:

  Numeric. Opinion on gun background checks.

- mctaxes:

  Numeric. Opinion on middle-class taxes.

- estate_tax:

  Numeric. Opinion on estate tax.

- raise_upper_tax:

  Numeric. Opinion on raising upper-income taxes.

- college:

  Numeric. Opinion on college affordability.

- abortion_waiting:

  Numeric. Opinion on abortion waiting period.

- abortion_never:

  Numeric. Opinion on banning abortion.

- abortion_conditions:

  Numeric. Opinion on abortion with conditions.

- late_term_abortion:

  Numeric. Opinion on late-term abortion.

- abortion_insurance:

  Numeric. Opinion on abortion insurance coverage.

- guaranteed_jobs:

  Numeric. Opinion on guaranteed jobs program.

- green_new_deal:

  Numeric. Opinion on Green New Deal.

- gun_registry:

  Numeric. Opinion on gun registry.

- immigration_separation:

  Numeric. Opinion on family separation policy.

- immigration_system:

  Numeric. Opinion on immigration system.

- immigration_wire:

  Numeric. Opinion on border wire.

- impeach_trump:

  Numeric. Opinion on Trump impeachment.

- israel:

  Numeric. Opinion on Israel policy.

- marijuana:

  Numeric. Opinion on marijuana legalization.

- maternityleave:

  Numeric. Opinion on maternity leave.

- medicare_for_all:

  Numeric. Opinion on Medicare for All.

- military_size:

  Numeric. Opinion on military size.

- minwage:

  Numeric. Opinion on minimum wage.

- muslimban:

  Numeric. Opinion on travel ban.

- oil_and_gas:

  Numeric. Opinion on oil and gas development.

- reparations:

  Numeric. Opinion on reparations.

- right_to_work:

  Numeric. Opinion on right-to-work laws.

- ten_commandments:

  Numeric. Opinion on Ten Commandments in schools.

- trade:

  Numeric. Opinion on trade policy.

- trans_military:

  Numeric. Opinion on transgender military service.

- uctaxes2:

  Numeric. Opinion on upper-class taxes (version 2).

- vouchers:

  Numeric. Opinion on school vouchers.

- gov_insurance:

  Numeric. Opinion on government health insurance.

- public_option:

  Numeric. Opinion on public option health insurance.

- health_subsidies:

  Numeric. Opinion on health insurance subsidies.

- path_to_citizenship:

  Numeric. Opinion on path to citizenship.

- dreamers:

  Numeric. Opinion on DACA/Dreamers.

- deportation:

  Numeric. Opinion on deportation policy.

- ban_guns:

  Numeric. Opinion on banning guns.

- ban_assault_rifles:

  Numeric. Opinion on banning assault rifles.

- limit_magazines:

  Numeric. Opinion on limiting magazine capacity.

- age:

  Numeric. Age in years (original column retained).

- gender:

  Numeric. Biological sex: `1` = Male, `2` = Female. Original integer
  column retained.

- census_region:

  Numeric. Census region.

- hispanic:

  Numeric. Hispanic origin (original column retained, used to derive
  `race_f4`).

- race_ethnicity:

  Numeric. Race/ethnicity code (original column retained, used to derive
  `race_f4`).

- household_income:

  Numeric. Household income.

- education:

  Numeric. Education level code (original column retained, used to
  derive `edu_f3`).

- state:

  Character. State of residence.

- congress_district:

  Numeric. Congressional district.

- weight:

  Numeric. Raked survey weight replicating the Nationscape weighting
  procedure (Newton-Raphson, 10 marginal dimensions, 5th/95th percentile
  trim). Pearson r = 0.996 vs. the original published Nationscape
  weight. All values are positive.

- wave_id:

  Numeric. Wave identifier.

- sex:

  Factor. Derived from `gender`: `1` = `"Male"`, `2` = `"Female"`, other
  = `NA`. Levels: `c("Male", "Female")`.

- age_f3:

  Factor. Derived from `age` using
  `cut(age, breaks = c(18, 35, 55, Inf), right = FALSE)`. Levels:
  `c("18-34", "35-54", "55+")`. `NA` for age \< 18 or missing.

- race_f4:

  Factor. Derived from `race_ethnicity` and `hispanic`: Hispanic origin
  takes precedence; `race_ethnicity %in% 3:14` = `"Other"` (collapses
  Asian); `race_ethnicity == 15` (some other race, non-Hispanic) = `NA`.
  Levels: `c("White", "Black", "Hispanic", "Other")`. Some `NA` values
  (respondents with "some other race", non-Hispanic).

- edu_f3:

  Factor. Derived from `education`: `1:3` = `"Less than HS"`, `4:7` =
  `"HS/Some college"`, `8:11` = `"College+"`. Levels:
  `c("Less than HS", "HS/Some college", "College+")`. No `NA` values.

- pid_f3:

  Factor. Derived from `pid3`: `1` = `"Democrat"`, `2` = `"Republican"`,
  `c(3, 4)` = `"Independent"` (code 4 = Other/No preference mapped to
  Independent per Nationscape methodology), other = `NA`. Levels:
  `c("Republican", "Independent", "Democrat")`.

- ns_region:

  Factor. Census region derived from `census_region`. Levels:
  `c("Northeast", "Midwest", "South", "West")`. No `NA` values.

- ns_hispanic:

  Factor. Three-category Hispanic ethnicity derived from `hispanic`: `1`
  = `"Not Hispanic"`, `2` = `"Mexican"`, `3:15` = `"Other Hispanic"`.
  Levels: `c("Not Hispanic", "Mexican", "Other Hispanic")`. No `NA`
  values.

- ns_race:

  Factor. Four-category race derived from `race_ethnicity`: `1` =
  `"White"`, `2` = `"Black"`, `4:14` = `"Asian/Pacific"`, `c(3, 15)` =
  `"Other"`. Levels: `c("White", "Black", "Asian/Pacific", "Other")`. No
  `NA` values.

- ns_age:

  Factor. Seven Nationscape age groups derived from `age`. Levels:
  `c("18-23", "24-29", "30-39", "40-49", "50-59", "60-69", "70+")`. No
  `NA` values.

- ns_language:

  Factor. Household language derived from `language`: `3` =
  `"English only"`, `1` = `"Spanish"`, `2` = `"Other"`. Levels:
  `c("English only", "Spanish", "Other")`. No `NA` values.

- ns_foreign_born:

  Factor. Country of birth derived from `foreign_born`: `1` =
  `"United States"`, `2` = `"Other"`. Levels:
  `c("United States", "Other")`. No `NA` values.

- ns_income:

  Factor. Nine household income brackets derived from `household_income`
  (codes `1`–`24`), plus `"No answer"` for `NA` responses. Levels:
  `c("<$20k", "$20-35k", "$35-50k", "$50-65k", "$65-80k", "$80-100k", "$100-125k", "$125-200k", ">=$200k", "No answer")`.
  No `NA` values.

- hh_income_f9:

  Factor. Nine-bracket household income harmonized with
  `cps_2023$hh_income_f9` for use as a `selection` variable in
  [`ipw()`](https://jdenn0514.github.io/surveywts/reference/ipw.md).
  Derived from `ns_income`; `"No answer"` is mapped to `NA`. Levels:
  `c("<$20k", "$20-35k", "$35-50k", "$50-65k", "$65-80k", "$80-100k", "$100-125k", "$125-200k", "≥$200k")`.

- ns_vote_2016:

  Factor. 2016 presidential vote derived from `vote_2016`: `1` =
  `"Trump"`, `2` = `"Clinton"`, `3:5` = `"Other"`, `6:8` = `"No vote"`
  (includes ineligible and don't recall). Levels:
  `c("Trump", "Clinton", "Other", "No vote")`. No `NA` values.

## Source

Derived from
[`surveycore::ns_wave1`](https://jdenn0514.github.io/surveycore/reference/ns_wave1.html).
See `data-raw/ns-wave1.R` for the construction script.

## See also

[gss_2024](https://jdenn0514.github.io/surveywts/reference/gss_2024.md),
[npors_2025_clean](https://jdenn0514.github.io/surveywts/reference/npors_2025_clean.md),
[cps_2023](https://jdenn0514.github.io/surveywts/reference/cps_2023.md)
