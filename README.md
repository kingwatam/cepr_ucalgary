# UCalgary research project (2024-2025)

Analysis code and documentation from my postdoctoral work at the Department of Oncology, University of Calgary, through the Cancer Epidemiology and Prevention Research (CEPR) program at Cancer Care Alberta.

The project examined late mortality and subsequent primary neoplasms among childhood cancer survivors in Alberta.

## At a glance

- **Cohort:** 2,581 childhood cancer survivors diagnosed in Alberta, 2001-2018
- **Follow-up:** to 31 December 2018
- **Outcomes:** all-cause and cause-specific mortality, and subsequent primary neoplasms
- **Reference populations:** Alberta mortality rates and Alberta Cancer Registry incidence rates
- **Methods:** Lexis-style survival analysis, competing-risks cumulative incidence, expected rates, standardized ratios, absolute excess rates, and Poisson regression
- **Tables and figures:** cohort description, standardized-rate tables, subgroup tables, heterogeneity and trend tests, and a five-panel cumulative-incidence figure
- **Implementation:** existing Stata pipeline adapted to the project, with new table and figure code written in R and reimplemented in Stata for the final outputs

This repository currently contains the README only. The analysis files and data are not included.

## Study background

The project examined late mortality and subsequent primary neoplasms among childhood cancer survivors diagnosed in Alberta between 2001 and 2018.

The cohort contains 2,581 survivors, with 408 deaths and 52 survivors who had at least one subsequent primary neoplasm. The analysis asks:

1. How much excess late mortality do childhood cancer survivors experience compared with the Alberta population?
2. How much excess subsequent-primary incidence is observed compared with Alberta population cancer incidence?
3. Do either depend on sex, first-primary group, age at diagnosis, diagnosis era, follow-up duration, attained age, health region or treatment?

The analysis is run for all survivors and for a five-year-survivor landmark population.

## Data sources and cohort construction

The analysis links three Alberta registry and administrative extracts: baseline cohort, mortality, and subsequent primary neoplasms.

Two population reference tables are used:

- Alberta age- and sex-specific mortality rates by cause, 1983-2019
- Alberta Cancer Registry incidence by site, age, sex and year

First primary tumours are classified using ICCC groups. Subsequent primaries are classified using ACR topography and behaviour codes. Forty-eight ACR categories are combined into 29 analysis categories so that reported groups have enough observed events.

A six-month washout is applied to exclude asynchronous or metachronous primaries. A separate landmark definition identifies survivors alive five years after diagnosis.

Linkage is performed on the registry person key plus a second longitudinal identifier. Merge checks confirm that no using-record was left unmatched when population reference tables were joined.

## Follow-up and time scale

The survival analysis uses a Lexis-style time scale.

- Time origin: date of birth
- Entry: diagnosis date, or the five-year landmark
- Exit: last known contact or death
- Administrative censoring: 31 December 2018

Three simultaneous splits separate the effects of attained age, calendar year and follow-up duration. The same breakpoints are implemented in R and Stata.

Entry and exit on the same date receive a small offset so that the record remains in the risk set. Leap-day diagnoses are adjusted for landmark entry.

The table and figure scripts use different time origins, which is documented in the code.

## Cause-of-death categories

Cause-of-death ICD-10 descriptions are parsed into analytical categories:

- Neoplastic
  - Recurrence or progression
  - Subsequent primary neoplasm
- Non-neoplastic
  - Health-related
  - External causes
- Suicide

Eight cause-specific rate types are used throughout the analysis. Recurrence and progression deaths have an expected rate of zero because population rates cannot be applied to them, so the zero-expected case is handled separately.

## Competing risks and expected events

Death from another cause is treated as a competing event when estimating cause-specific cumulative incidence. The analysis reports cumulative incidence, confidence intervals, risk tables, and observed and expected curves.

Expected events are calculated from the Alberta population rate tables using each survivor's age, sex and calendar period. Expected cumulative incidence is derived by accumulating the expected hazard across each follow-up interval.

The analysis reports:

- Standardized mortality ratios
- Standardized incidence ratios
- Absolute excess rates per 10,000 person-years
- Person-years of follow-up
- Exact Poisson confidence limits for observed-to-expected ratios
- Normal-approximation limits for absolute excess rates

Negative or undefined excess rates are suppressed where necessary.

## Regression models and p-value grid

Two Poisson model families are used for each stratifying variable:

1. Observed counts with the log of the expected count as an offset, for standardized mortality or incidence ratios
2. Excess counts with person-time as the offset, for absolute excess rates

For each variable the code tests:

- Heterogeneity across categories, using a likelihood-ratio test against an intercept-only model
- Linear trend, also using a likelihood-ratio test

The grid covers 11 stratifying variables and 8 cause-specific rate types in two analysis populations, producing several hundred likelihood-ratio tests.

The same model structure is implemented in R and Stata. Results are written to formatted tables and pasted into manuscript-ready workbooks.

## Table generation

### Cohort description

Table 1 describes all survivors, survivors who died, survivors with a subsequent primary, and survivors without a subsequent primary. Results are shown for all survivors and for five-year survivors.

The Stata version uses `collect` and `dtable` with custom median and interquartile-range formatting. The R version uses a table builder designed for the same layout.

### Standardized-rate and subsequent-primary tables

Tables report observed counts, expected counts, rates, standardized ratios, absolute excess rates and confidence intervals for each rate type and stratifying variable. Separate p-value tables report heterogeneity and trend tests.

Subsequent primaries are summarized by site and by patient characteristics. A record is suppressed when a repeat subsequent primary occurs at the same site as an earlier tumour in the same survivor, and the difference in counts is documented in the code.

Stata frames are used to hold the results for each table. In R, custom table functions format the model output and write it to Excel or the clipboard for transfer into manuscript tables.

## Figure generation

The final publication figure has five panels:

- A. All-cause mortality
- B. Recurrence or progression
- C. Subsequent primary neoplasm
- D. Non-neoplastic mortality
- E. Subsequent primary incidence

Each panel contains observed step curves, 95% confidence bands, the expected-population curve, values annotated at five, ten and fifteen years, and a shared legend. The Stata version uses `stcompet` for cumulative incidence, `stexpect` for expected incidence, risk tables, and `grc1leg2` to assemble the panels with a common legend. The R version uses `cuminc`, `ggcuminc`, `patchwork` and `ggsave`.

The final outputs include a 600-dpi raster image and a vector version.

## R and Stata cross-checks

The underlying survival and rate calculations were adapted from existing Stata code developed from the study team's earlier PhD and project work. I built an R implementation for this project and wrote new table and figure code in R. After extended checking did not fully reconcile the R and Stata implementations, I adapted the existing Stata code to the project data and used it for the final publication tables and figures.

The R version covered data preparation, core table construction, Lexis splitting, competing-risks calculations and draft figures. The Stata version produced the final tables, expected-incidence calculations, risk tables, figure annotations and manuscript outputs.

The two implementations share the time-origin concept, split points, population-rate merge keys, competing-event coding, outcome definitions and excess-count model structure.

R exports an analysis-ready dataset for Stata, and the Stata output is imported back into R for comparison. Stata date origins, variable names and data types are handled explicitly.

A clinically important classification was compared between the two implementations: whether a death should be attributed to a subsequent primary rather than recurrence. A discrepancy led to a date-comparison problem being identified and corrected in both implementations.

## Quality-control conventions

- Population-rate joins check that no using-record was left unmatched.
- Cohort counts are checked after construction.
- Differences in counts are explained in comments rather than silently corrected.
- Result stores use create-or-replace guards so scripts can be rerun.
- Failures within the model grid are handled per cell so that one non-convergent model does not stop the full analysis.
- The Stata files record the project, principal investigator, author, date, software version and purpose.
- The public repository will contain documentation only. It will not contain row-level registry data, direct identifiers, credentials, internal paths, manuscripts or journal-submission records.

## Analysis library

The helper library originated at HKU as an attempt to recreate common Stata functions in R. It was extended at CUHK with reporting, psychometric and model-summary functions, and again for this project with date and interval helpers, a faster EQ-5D implementation and publication-table functions.

Functions used in this project include Stata-style summaries and cross-tabulations, table generation, side-by-side model tables, multi-sheet Excel output, age recoding, date and interval calculations, and function importing across scripts.

## Related publication

Tam KW, Pitt TM, Reynolds K, Spavor M, Truong TH, Giles J, Guilcher GM, Logie N, Rahamatullah I, Schulte F, Fidler-Benaoudia MM. Subsequent Primary Neoplasms and Mortality Among Survivors of Childhood Cancer in Alberta, Canada. *Cancers*. 2026;18(4):694.

https://doi.org/10.3390/cancers18040694
