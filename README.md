# Software Quality Insight — project research and roadmap

For a short step-by-step explanation of the project, start with [ABOUT.md](ABOUT.md).
For folder purposes and the step-by-step run/completion checklist, see [TODO.md](TODO.md).

## Project purpose

This project follows **Use Case 28: Software Quality Insight** from the DS Sprint Skill Based Assessment 2 brief. It investigates which software module and process metrics are associated with defect counts, then turns those findings into practical priorities for code review and testing.

The brief is an assessment specification, not a dataset. It names five expected source files (`modules.csv`, `metrics.csv`, `defects.csv`, `commits.csv`, and `releases.csv`) and their expected fields, but those files were not included with the supplied PDF. Do not assume they exist or invent their contents. The analysis must start with the instructor-provided CSVs if available, or use the documented public-data fallback below.

## Research summary and dataset decision

### What the assignment requires

The target analysis unit should be **one module in one release**. The brief's questions require, at minimum:

| Concept | Required information | Intended analysis |
|---|---|---|
| Module | Stable module/file identifier, component, language, size | Defects by module/component; normalize for size |
| Product metrics | Complexity, coupling, test coverage | Compare metrics with defect counts |
| Defect outcome | Defect count, severity, release | Counts, repeated defects, severity mix, release trend |
| Process metrics | Commit count, contributors, changed lines/churn | Compare development activity with defects |
| Release | Version and date linked to the module | Trend over time and correct join grain |

The PDF's data-quality checklist also requires checks for missing values, duplicate module-release records, inconsistent names, extreme complexity, and defect rows that do not match the module/release data.

### Recommended selection order

1. **Use the instructor's five CSVs if they are available.** They are the intended source and should match the assessment's requested columns most closely. Confirm their primary keys and row grain before joining.
2. **If they are unavailable, use the prepared SmartSHARK release-level replication dataset as the public fallback.** Its Zenodo record describes labeled data for 398 releases and provides a reusable `release-level-data.tar.gz` archive (about 616 MB). The associated study describes file-level release data, static code metrics, change/churn features, and defect counts; issue metadata includes severity. This is a closer fit to a module-release study than a commit-only dataset. See [the Zenodo replication package](https://zenodo.org/records/5675024) and [the study's methods and dataset description](https://link.springer.com/article/10.1007/s10664-021-10092-4).
3. **Do not use ApacheJIT as the primary dataset for this assignment.** It is a useful commit-level dataset with commit metrics and a `buggy` label, but its unit is commits and its label is binary; that does not directly answer module-by-release defect-count and severity questions. See [ApacheJIT's data description](https://github.com/hosseinkshvrz/apachejit) and [the paper](https://arxiv.org/abs/2203.00101).
4. **Use PROMISE/tera-PROMISE datasets only after a column-by-column audit.** They are established software defect research resources, but many benchmark tables have binary defect labels or static metrics without the release/process fields required here. See the [tera-PROMISE repository](https://openscience.us/repo/) and the [PROMISE repository information](https://promisedata.org/?cat=131).

### Fit and limitations of the SmartSHARK fallback

| Assignment concept | SmartSHARK fallback fit | How to handle it |
|---|---|---|
| Module and release | Strong: prepared release-level data is file-level and covers many releases | Use file/class as the module proxy; retain project and release identifiers |
| Complexity/coupling/size | Available: WMC, CBO, and file LOC fields were confirmed in the Calcite tables | Use the selected, interpretable measures; the source has 4,280 columns, most outside this assignment's scope |
| Churn / changed lines | Available: MOSER revisions, summed additions, summed deletions, and signed net code change | Use additions plus deletions as a change-volume proxy. Keep signed net change separate; it can be negative. |
| Defect counts | Strong: the paper describes per-file bug counts | Preserve zero counts; check how the labels assign defects to releases |
| Defect severity | Available through linked issue metadata | Deduplicate issue IDs per file-release; resulting severity total reconciles to the prepared defect count |
| Contributors / commit counts | Available as process proxies: distinct file authors and file revisions | Label `MOSER_authors` as contributor count and `MOSER_revisions` as revision-count proxy, not a separate commit table |
| Test coverage | Not available in the selected Calcite data | Show as unavailable; do not make a coverage comparison or impute zero. Use instructor data if coverage is required. |
| Components | Derived from Java package path | Use top-level package as an analysis component; preserve the full module path in the row-level data |

The SmartSHARK public archive is a sizeable download, so the project should use only the prepared release-level archive needed for the coursework, not the full raw repository database. The Zenodo page explicitly distinguishes the reusable `release-level-data.tar.gz` from a similarly sized archive marked `DO_NOT_USE`; use the former only. The SmartSHARK data-release page documents broader source contents and licenses for the separately distributed database: [SmartSHARK data releases](https://smartshark.github.io/dbreleases/).

### Implemented Calcite subset

The selected subset contains **19,751 module-release rows**, **1,644 distinct Java files**, **24 package components**, and **16 releases** from 1.0.0 through 1.15.0. There are 2,138 prepared defect labels, including 1,577 module-release rows with at least one defect. Severity-linked issue counts reconcile with the defect labels after issue identifiers are deduplicated within each file and release. There are no duplicate module-release keys or unmatched severity file paths in this subset.

The signed `MOSER_sum_code_churn` is the sum of added lines minus deleted lines, so negative values are legitimate. For the coursework's change-activity comparison, the project derives `change_volume_proxy = MOSER_sum_lines_added + MOSER_sum_lines_deleted`. This remains a proxy rather than an exact commit-level churn measure. The dataset has no test-coverage field or calendar release date. Release charts therefore use version order. See `data/processed/data_dictionary.csv`, `data/processed/cleaning_log.csv`, and `data/processed/data_quality_report.json` for field mappings and audit results.

Descriptive Spearman associations with defect count are positive for WMC complexity (rho 0.261), CBO coupling (0.266), change volume (0.229), revision count (0.234), contributor count (0.222), and file size (0.276). Each is exploratory; zero-heavy defect counts and repeated files across releases limit statistical interpretation. The highest cumulative defect counts are in `SqlToRelConverter.java` (86 across 16 releases), `RexImpTable.java` (68 across 16), and `SqlValidatorImpl.java` (66 across 16). These are review candidates, not findings of causation.

### Research cautions that affect the analysis

- A defect count is not automatically comparable across modules of very different sizes. Report raw counts and an exposure-adjusted measure such as defects per thousand lines of code (KLOC), when size units are known and valid.
- A dataset's defect labels are imperfect measurements. A study of defect-data collection reports substantial label noise and explains that release-level and commit-level datasets use different labeling assumptions. Describe the chosen dataset's labeling method and avoid causal claims. See [Herbold et al., dataset collection study](https://link.springer.com/article/10.1007/s10664-021-10092-4).
- Use the term **association** for descriptive correlations. A correlation between complexity and defects does not show that complexity caused the defects.
- Avoid leakage if the work later adds prediction: predictors must be available at the time the prediction would be made. Do not use post-release defect information or features calculated after the target release as predictors.
- Do not delete extreme complexity values until they are checked for valid but rare modules, unit mistakes, or data errors. Show robust summaries or use a documented transformation if skew is severe.
- Do not replace missing coverage with zero. Missing means unknown unless the data dictionary establishes otherwise.

## Round-by-round execution plan

Use the rounds in order. Keep the same cleaned analysis table as the single source of truth for the notebook, workbook, and presentation.

### Round 1 — Quiz (20 marks)

**Topics to prepare:** data analytics concepts; data preprocessing; exploratory data analysis; feature engineering; Excel; NumPy; Pandas; Matplotlib; Seaborn.

**Checklist**

- [ ] Review descriptive statistics, distributions, correlation, sampling, and interpretation of charts.
- [ ] Review missing values, duplicates, data types, outliers, joins, and encoding.
- [ ] Review Pandas selection, grouping, aggregation, merge, and missing-value operations.
- [ ] Review NumPy arrays and basic numerical summaries.
- [ ] Review Matplotlib/Seaborn chart selection and chart interpretation.
- [ ] Review Excel formulas, PivotTables, PivotCharts, and summary sheets.
- [ ] Complete the quiz and save the score sheet.

**Evidence:** completed MCQ test and score sheet.

### Round 2 — EDA and feature engineering (30 marks)

**Goal:** produce a trustworthy, documented module-release dataset and answer the six use-case questions descriptively.

**Step 1: inventory and understand the source**

- [x] Record the data source, version/DOI, license, retrieval date, and citation.
- [x] List every file/table, its row count, fields, data types, and stated grain.
- [x] Identify stable module, component/project, release, and defect identifiers.
- [x] Create a data dictionary with units and field definitions; mark unavailable fields explicitly.

**Step 2: audit quality before cleaning**

- [x] Count missing values by field and inspect why they are missing.
- [x] Check duplicate rows and duplicate module-release keys.
- [x] Standardize names through an explicit mapping table; retain original names.
- [x] Check unmatched module/release/defect records before and after joining.
- [x] Inspect numeric ranges, units, impossible values, and extreme complexity observations.
- [x] Keep a cleaning log recording each change and its reason.

**Step 3: form the analysis table**

- [x] Set the intended grain to one row per module-release.
- [x] Aggregate multiple commits and defects to that grain with documented rules.
- [x] Join tables using verified keys, not row order or fuzzy names alone.
- [x] After each join, verify row counts and ensure defect/commit counts did not multiply.
- [x] Save a raw, cleaned, and analysis-ready version separately.

**Step 4: engineer justified features**

- [x] Compute defects per KLOC only when size and units support it.
- [x] Derive churn features from changed lines only when their definition and time window are known.
- [x] Add repeated-defect indicators only with a stated threshold or release-history rule.
- [x] Encode categorical fields for analysis when required; preserve readable labels for charts.
- [x] Exclude IDs and post-outcome fields from any predictive feature set.
- [x] Use feature selection or dimensionality reduction only if the number of features and analysis goal justify it.

**Step 5: answer the assignment questions**

- [x] Which modules have more defects? Show raw counts and, where valid, size-normalized rates.
- [x] How does complexity relate to defects? Use scatter/box plots and an appropriate correlation measure.
- [x] Does code churn relate to quality? Compare churn with counts/rates, accounting for module size where possible.
- [x] How does test coverage differ? Analyze only if real coverage data exists; otherwise document it as unavailable.
- [x] Which components show repeated defects? Define “repeated” and show component/release evidence.
- [x] Does contributor count relate to defect frequency? Use a real contributor field or a documented derivation; otherwise state the gap.

**Evidence:** cleaned dataset, EDA notebook, key visualizations, engineered features, selected features if appropriate, and data-quality notes.


### Round 3 — Excel analysis (30 marks)

**Goal:** make the findings usable and auditable by a nontechnical reviewer.

**Suggested workbook sheets**

The completed workbook uses `Read Me`, `Summary`, `Module Ranking`, `Component Trend`, `Release Trend`, `Severity`, and `Analysis Data` sheets. The analysis table is filterable and ready for Excel's Insert > PivotTable workflow. It includes grouped summaries and charts, but this exported workbook does not contain native interactive PivotTables because the authoring tool's PivotTable export is unreliable.

**Checklist**

- [x] Import the cleaned analysis table and retain stable IDs.
- [x] Format headings, dates, counts, percentages, and missing values consistently.
- [x] Use formulas/statistical functions that can be inspected and traced.
- [ ] Create native interactive PivotTables/PivotCharts in Excel. The workbook includes a filterable source table and grouped summaries/charts; native pivots remain an Excel-only follow-up because the workbook authoring export does not reliably preserve them.
- [x] Create charts that directly answer the required questions.
- [x] Add a summary sheet with the most important evidence and recommendations.
- [x] Reconcile totals and row counts against the notebook before finalizing.

**Evidence:** formatted workbook with formulas, statistical analysis, grouped summary tables, editable charts, a filterable source table, and a summary sheet. Native interactive PivotTables/PivotCharts remain an Excel follow-up listed in Implementation status.


### Round 4 — Business-insight pitch (20 marks)

**Goal:** recommend where code review and testing effort should go, based on evidence.

**Suggested short deck**

1. Business question and why module quality matters.
2. Dataset, unit of analysis, time span, and limitations.
3. Most defect-prone modules/components (with size-normalized context).
4. Metric relationships: complexity, churn, coverage if available, and contributors if available.
5. Repeated-defect and release trends.
6. Prioritized actions for code review/testing, each tied to a chart or statistic.
7. Caveats and next steps for data collection.

**Checklist**

- [x] State findings in plain language and support each with a number or chart.
- [x] Separate observed association from causal explanation.
- [x] Avoid recommending priorities based only on raw defect totals where module size differs.
- [x] Explain missing or incomplete fields, especially coverage and contributor counts.
- [x] Make recommendations actionable (which modules/components, what review/testing action, why).
- [x] Keep the pitch concise and consistent with the notebook and workbook.

**Evidence:** presentation slides and a clear, evidence-based business pitch.


## Project completion checklist

- [x] Dataset provenance and license recorded.
- [x] Data dictionary and data-quality log complete.
- [x] Joins verified at module-release grain.
- [x] Cleaned dataset saved with no unexplained duplicate keys.
- [x] Notebook runs top-to-bottom and its figures agree with reported results.
- [x] Excel figures and totals reconcile with the notebook.
- [x] Slides use the same definitions and numbers as the analysis.
- [x] Limitations and recommendations are explicit.
- [ ] All assessment evidence is complete. The available project files are present; the course quiz score sheet and native interactive PivotTables/PivotCharts are still outstanding.


## Suggested project folder layout

```text
software-quality-insight/
├── README.md
├── data/
│   ├── raw/                 # Original, unmodified source data
│   ├── interim/             # Imported and normalized tables
│   └── processed/           # Final module-release analysis table
├── notebooks/
│   └── software_quality_eda.ipynb
├── excel/
│   └── software_quality_analysis.xlsx
├── presentation/
│   └── business_insights.pptx
├── figures/
└── references/
    └── sources.md
```

Do not commit large raw dataset archives unless the course submission requires them. Keep the source URL, version, checksum if available, license, and download/retrieval date in `references/sources.md`.

## Research references

- Assignment specification: supplied `CH28.pdf`, “Use Case 28 — Software Quality Insight,” including data schema, data-quality conditions, six analysis questions, round rubrics, and deliverables.
- Herbold, S., Trautsch, A., Trautsch, F., & Ledel, B. (2021). [Problems with SZZ and Features: An empirical study of the state of practice of defect prediction data collection](https://link.springer.com/article/10.1007/s10664-021-10092-4). Empirical Software Engineering. Dataset methods and noise limitations.
- Herbold et al. [Replication package and release-level data](https://zenodo.org/records/5675024). DOI: 10.5281/zenodo.5675024. Use the archive `release-level-data.tar.gz`; do not use the archive explicitly named `DO_NOT_USE_release-level-data.tar.gz`.
- [SmartSHARK data releases](https://smartshark.github.io/dbreleases/). Ecosystem descriptions, contents, release versions, and licensing information.
- Keshavarz, H., & Nagappan, M. (2022). [ApacheJIT: A Large Dataset for Just-In-Time Defect Prediction](https://arxiv.org/abs/2203.00101). Alternative considered; commit-level unit and binary bug label do not directly match this assignment's module-release count questions.
- [ApacheJIT dataset repository](https://github.com/hosseinkshvrz/apachejit). Dataset file descriptions and replication materials.
- [tera-PROMISE repository](https://openscience.us/repo/) and [PROMISE repository information](https://promisedata.org/?cat=131). Alternative sources to audit if the instructor data is unavailable.

## Implementation status

- [x] Rechecked the roadmap against the supplied assessment PDF. The four rounds, six analytical questions, Excel requirements, pitch focus, and final deliverables are represented above.
- [x] Selected and downloaded the SmartSHARK prepared release-level archive because the instructor CSVs were not included with the supplied PDF. Its Zenodo checksum matches the published MD5.
- [x] Inspected the archive and documented field mappings, license, definitions, and limitations.
- [x] Built the cleaned Calcite module-release dataset, severity table, summary tables, figures, data dictionary, cleaning log, and quality report.
- [x] Created the EDA notebook and confirmed its code cells run through the analysis.
- [x] Created the Excel workbook with formula-based summaries, filterable analysis data, grouped component/release/severity summaries, and charts.
- [x] Created the six-slide business pitch with editable charts and tables.
- [ ] Take the Round 1 quiz and save the score sheet. The brief gives topics and marks but does not include quiz questions or a score sheet.
- [ ] Create native interactive PivotTables/PivotCharts in Excel. The workbook's Analysis Data table is ready to use as the PivotTable source.

The public fallback directly supports module-release defect, complexity, coupling, change activity, contributor, component, and severity analyses. It does not provide test coverage or calendar release dates, so those comparisons cannot be completed from this source. Replace the fallback with the instructor's five CSVs if they become available and remap the analysis before submission.
