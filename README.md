# Vanguard Digital Experience Analysis: A/B Test of a New Online Process

*Team "Guardians of the Vanguard" · Ironhack Data Analytics, Module 2 project · Motto: **show the numbers, then decide.***

## 1. Project Overview

Vanguard, an investment company, redesigned the online process its clients use to complete a task on its website. The process has five recorded stages: `start → step_1 → step_2 → step_3 → confirm`. Vanguard ran an A/B test to find out whether the new design helps clients finish the process.

**Business problem.** Does the new digital experience help more clients complete the journey, and does it do so without creating new problems (confusion, restarts, more calls to customer care)?

**What changed.** Clients were split into two groups:

- **Control:** the old design (the baseline).
- **Test:** the new design (the variation being evaluated).

**Main objectives of the analysis**

- Compare Control and Test on completion and drop-off at every step.
- Check whether each difference is real or could be chance, using hypothesis tests.
- Look at time, restarts, navigation difficulty and customer care calls.
- Separate statistical significance from practical, business relevance.
- Give Vanguard a recommendation it can act on.

**Storytelling approach.** We presented the results through "Skeptical Sam", a cautious new investor with little money to invest who only believes numbers. Each result starts with Sam's doubt, then the finding, then Sam's reaction.

**Final Presentation:** *Add the link to the Canva deck here.*
**Interactive dashboard (Tableau Public):** *Add the link here.*

## 2. Research Questions

1. Does the new design improve completion? Is the difference statistically significant?
2. At which step do clients drop off, and does the drop-off differ between Test and Control?
3. Do the visit-level and the client-level views tell the same story?
4. Is the new design faster? Which step takes the longest, and does age or gender change the picture?
5. Do clients restart the process more often, or show signs of navigation difficulty, in the new design?
6. Does the new design lead to more customer care calls? Do clients who call finish more often?
7. Which clients (high balance, long tenure) rely on calls rather than the website?
8. Do the differences give enough evidence to support a business decision?

## 3. Dataset Description

### Data sources

| File | Content | Size |
|---|---|---|
| `df_demo_clean.csv` | Client demographics and account information | 70,595 clients |
| `roster_clean.csv` | Which clients were in the experiment, and their group (`variation`: Test or Control) | 50,500 clients |
| `final_web_data_merged.csv` | Web events: one row each time a client reached a step | 744,641 events |

### Main variables

| Variable | Meaning |
|---|---|
| `client_id` | Unique client |
| `visitor_id`, `visit_id` | Unique visitor and visit (one client can have several visits) |
| `process_step` | `start`, `step_1`, `step_2`, `step_3` or `confirm` |
| `date_time` | Timestamp of the event |
| `variation` | `Test` (new design) or `Control` (old design) |
| `clnt_tenure_yr`, `clnt_tenure_mnth` | How long the person has been a client |
| `clnt_age`, `gendr` | Age and gender (`U` = unspecified) |
| `num_accts`, `bal` | Number of accounts and total balance |
| `calls_6_mnth`, `logons_6_mnth` | Customer care calls and online logons in the last 6 months |

### Merged tables used in the analysis

- `df_clients`**:** roster joined to demographics (inner join), 50,488 clients.
- `final_merge.csv`**:** web events joined to `df_clients`, 317,135 event rows, 14 columns. Used for most analyses (funnel, calls, age, gender, balance).
- `df_events.csv`**:** web events joined to the roster only, 317,235 event rows, 6 columns. Used for time and navigation analyses.

### Relationships

`roster_clean` and `df_demo_clean` join on `client_id`. The web events join to clients on `client_id`. One client can have several visits, and one visit has several events (one per step reached).

### Main Data Quality Issues

- **Clients did not match across files.** Demographics hold 70,595 clients, the roster 50,500. Only clients present in the roster have a Test or Control label.
- **Most web events had no experiment group.** Of 744,641 events, only about 317,000 belong to clients in the experiment.
- **Possible incomplete journeys.** Some visits never reach `confirm`, and some visits skip a step or repeat one.
- **Extreme time values.** Some steps last 17,000–18,000 seconds, most likely a page left open and not real working time.
- **Unspecified gender.** Part of the clients have `gendr = U`.

### Data Cleaning and Preparation

- Cleaned the demographics and roster files (`*_clean.csv`) and merged them with inner joins, so every analysed client has a group and demographic data.
- Kept the original files unchanged, and saved the processed tables as separate files.
- Counted each client once, at their furthest recorded step, for client-level analyses. Counted each visit once, at its furthest step, for visit-level analyses.
- Treated extreme step times as outliers for the time analysis (see Limitations).

> **Important decision:** the inner joins restrict the analysis to clients who appear in the roster and in the web data (50,488 clients: 26,961 Test and 23,527 Control). Results describe those clients only.

## 4. Methodology

**Exploratory Data Analysis.** We checked the size, structure and missing values of each file, the balance between Test and Control, the distribution of ages, balances and calls, and the number of visits and events per client.

**User Journey Analysis.** We built the path of each visit through the five steps. We measured how far each visit or client got, where they stopped, the time per step, and restarts (going back to `start`).

**Performance Metrics (KPIs).**

| KPI | Why it fits |
|---|---|
| Completion rate (client reaches `confirm`) | Direct measure of the business goal |
| Drop-off rate per step | Shows where the process loses clients (first priority) |
| Time to complete and time per step | Measures how easy the process is |
| Restarts and navigation difficulty indicators | Signals of confusion or friction |
| Average calls per client; completion with and without calls | Shows load on customer care and how calls relate to completion |

**Drop-off definition.** For each step, the share of visits (or clients) whose furthest recorded step was that step, among those whose furthest step was that step or later.

**Group Comparison.** Every KPI was calculated separately for Test and Control, then compared.

**Statistical Analysis.**

| Question | Test | Significance level (α) | Why |
|---|---|---|---|
| Drop-off per step, clients and visits | Two-proportions z-test | 0.05 / 4 = **0.0125** (Bonferroni, four steps) | Comparing two proportions; the correction avoids false positives from running four tests |
| Average calls per client | Welch's t-test | 0.05 | Comparing two means without assuming equal variances |
| Completion with vs without calls, per design | Two-proportions z-test (one-sided) | 0.05 / 2 = **0.025** | Two tests run, one per design |
| Time per step, by age and gender | Welch's t-test | 0.05 | Comparing means of groups of different size |
| Restart rate | Z-test and chi-square test | 0.05 | Comparing proportions of visits |

**Hypotheses.** H0 says there is no real difference between Test and Control. We reject H0 only if the p-value is below α. Rejecting H0 means the difference is real, whether it is good or bad for the new design. Not rejecting H0 means we do not have enough evidence of a difference. It does not prove that there is none.

**Assumptions.** Clients are independent. Visit-level p-values should be read with caution, because one client can have several visits. We used the client-level analysis as the main one for that reason.

## 5. Key Findings

### Completion

| | Test | Control |
|---|---|---|
| Clients who reached `confirm` at least once | 69.3% (18,682 of 26,961) | 65.6% (15,429 of 23,527) |

Test is about 3.7 percentage points higher (z = 8.9, p < 0.001). *Calculated from* `final_merge.csv`*. Please check this against the team notebooks before submitting.*

### Drop-off per step, clients (α = 0.0125)

| Step | Test | Control | Difference | p-value | Decision |
|---|---|---|---|---|---|
| start | 9.10% | 13.96% | −4.86 pp | < 0.001 | Reject H0: Test is better |
| step_1 | 8.08% | 7.19% | +0.89 pp | 0.0004 | Reject H0: **Test is worse** |
| step_2 | 6.26% | 6.73% | −0.47 pp | 0.053 | Not enough evidence |
| step_3 | 11.53% | 11.94% | −0.41 pp | 0.213 | Not enough evidence |

### Drop-off per step, visits (α = 0.0125)

| Step | Test | Control | Difference | p-value | Decision |
|---|---|---|---|---|---|
| start | 13.66% | 23.48% | −9.82 pp | < 0.001 | Reject H0: Test is better |
| step_1 | 12.17% | 14.16% | −2.00 pp | < 0.001 | Reject H0: Test is better |
| step_2 | 9.12% | 9.63% | −0.51 pp | 0.052 | Not enough evidence |
| step_3 | 15.02% | 15.93% | −0.92 pp | 0.0078 | Reject H0: Test is better |

**Where visits and clients disagree.** At step_1 the visit view favours Test, while the client view favours Control. We flag it and do not claim a cause.

### Time, restarts and navigation

- **Time:** the median completion time is 251 seconds in Test against 275 in Control. Test is faster in every age group and for both genders (Welch's t-test, p < 0.05). The longest step is step_3, about 105 seconds on average.
- **Restarts:** Test restarts about 77% more than Control. Most of the extra restarts come from step_1 (z-test and chi-square test, p < 0.05).
- **Possible navigation difficulty:** 59.37% of Test clients show at least one indicator, against 52.81% in Control (significant).

### Customer care calls

- **Average calls per client:** 3.06 (Test) against 3.13 (Control). The difference is −0.067 calls (Welch's t-test, p = 0.0006). It is statistically significant but very small in practice.
- **Completion with and without calls (α = 0.025):**

| Design | Zero calls | At least one call | Difference | Decision |
|---|---|---|---|---|
| Control | 62.79% | 66.07% | +3.28 pp | Reject H0 |
| Test | 65.48% | 70.00% | +4.52 pp | Reject H0 |

This is an association. The data does not tell us why clients called.

### Other questions

*The notebooks also cover high-balance clients with many calls but few logons, and tenure against balance. Add one line per result here once the team confirms the numbers.*

### Statistical vs practical relevance

- The drop-off at the start (about 5 points lower for clients) and the higher completion rate are large enough to matter for the business.
- The difference in average calls (0.07 calls) is real but too small to change customer care workload.
- The step_1 result is small (under 1 point) but points against the new design, and it comes with more restarts. That makes it worth a closer look.

## 6. Conclusions

- **What the experiment shows.** The new design helps clients get past the start of the process, increases completion and is faster. It does not add customer care calls.
- **Statistical support.** The start drop-off, completion rate, time, and the call result are supported by hypothesis tests. Step_2 and step_3 show no clear client-level difference.
- **A warning sign.** Step_1 drop-off is higher in Test for clients, restarts are higher, and more Test clients show signs of navigation difficulty.
- **Limitations.**
  - A/B test results show association between design and outcome. The calls analysis cannot show why clients called.
  - Only clients present in both the roster and the web data are included (50,488).
  - Visit-level tests treat visits as independent, but one client can have several visits.
  - Extreme time values (idle pages) may affect average times.
  - The data covers one period, so seasonal effects are not tested.
- **Recommendation (draft, to confirm as a team).** Keep the new design, because it improves completion and speed. Before a full rollout, investigate and fix step_1, where clients drop off more and restart more, and re-test.

## 7. Next Steps

- Investigate step_1 (screens, wording, buttons) and run a follow-up test.
- Explain why clients restart more in Test, using the navigation indicators.
- Test additional segments: age groups, balance levels and tenure.
- Check time of day, weekday and season effects.
- Check for clients who confirm more than once, and treat it as a separate issue.
- Collect the reason for customer care calls, to tell whether calls help clients or signal confusion.

## 8. Repository Structure

```text
├── data/
│   ├── raw/           # Original datasets
│   └── clean/         # Processed datasets (df_demo_clean, roster_clean, final_web_data_merged, final_merge, df_events)
├── notebooks/         # Jupyter notebooks (merge, hypotheses, time, restarts, age and gender, balance)
├── figures/           # Charts and visualizations
├── slides/            # Final presentation or link
├── config.yaml        # Project file paths
├── pyproject.toml     # Python dependencies
└── uv.lock            # Locked dependency versions
```

Each directory contains its own README with additional information about file organization and naming conventions.

## 9. Team Members

| Name                  | Role           | Main Contributions                                                                                                                       |
| --------------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Bindu Madhavi Valmiki | *Data Analyst* | Drop-off per step (visits and clients) with hypothesis tests, average calls, completion by calls, presentation opening and Sam storyline |
| Juan                  | *Data Analyst* | Time to complete, longest step, restarts, age and gender analysis                                                                        |
| Jacob                 | *Data Analyst* | High-balance clients with many calls and few logons, tenure vs balance, navigation errors, key findings and recommendations              |
