# Software Quality Insight — run guide and TODO

Use this file as the practical checklist for opening, understanding, and completing the project. The project overview, assignment interpretation, dataset research, and analysis limitations are in [README.md](README.md).

## 1. What each folder is for

| Folder | Use | What to do with it |
|---|---|---|
| `data/raw/` | Original downloaded source archive, kept unchanged for provenance. | Keep as the source copy. It is a large archive; the notebook does not need to open it during a normal run. |
| `data/interim/` | Extracted, source-specific files prepared before the final analysis table. | Use to inspect how the selected Calcite data was extracted and which source files contributed. Do not edit these as your analysis output. |
| `data/processed/` | Clean, analysis-ready tables, summaries, data dictionary, and quality report. | These are the inputs used by the notebook and the handoff point for checking or reusing the analysis. |
| `notebooks/` | The exploratory analysis notebook. | Run the notebook from top to bottom to reproduce the displayed analysis using `data/processed/`. |
| `figures/` | Exported charts used in the analysis and presentation. | Open the PNGs directly, or use them when updating the report or slides. |
| `excel/` | Excel workbook with the analysis data, summaries, and charts. | Open in Excel for filtering and review. The analysis table is prepared for pivoting; native interactive PivotTables still need to be added if the assessment requires them. |
| `presentation/` | Business-facing slide deck summarizing the findings. | Open in PowerPoint and review the claims against the notebook and processed data before submission. |
| `references/` | Source archive inventory and source notes/links. | Use to trace dataset provenance and cite the data source. |

### Important files

- `data/processed/calcite_module_release.csv` — main module-by-release analysis table.
- `data/processed/calcite_issue_severity_long.csv` — issue/severity detail used for severity analysis.
- `data/processed/module_defect_summary.csv`, `component_summary.csv`, and `release_summary.csv` — grouped analysis outputs.
- `data/processed/data_quality_report.json`, `cleaning_log.csv`, and `data_dictionary.csv` — quality checks, cleaning record, and field definitions.
- `notebooks/software_quality_eda.ipynb` — runnable analysis notebook.
- `excel/software_quality_analysis.xlsx` — review workbook.
- `presentation/business_insights.pptx` — presentation deliverable.
- `references/sources.md` and `references/archive_members.csv` — source citations and archive inventory.

## 2. Run the prepared analysis

The prepared outputs are already included, so you can review the workbook and slides without running anything. To rerun the exploratory analysis:

1. Open a terminal in the `software-quality-insight` project folder.
2. Install Python 3 and Jupyter Notebook (or use an existing Anaconda/Jupyter environment).
3. Install the notebook libraries listed in `requirements.txt` if they are not already available:

   ```bash
   python -m pip install -r requirements.txt
   ```

4. Start Jupyter from the project folder:

   ```bash
   jupyter notebook
   ```

5. Open `notebooks/software_quality_eda.ipynb` and choose **Run All**. Run from the project root so the notebook's relative paths can find `data/processed/`.
6. Review the notebook output and compare its counts and charts with the CSV summaries and the workbook.
7. Open `excel/software_quality_analysis.xlsx` in Excel and `presentation/business_insights.pptx` in PowerPoint to review those deliverables.

The notebook reads the prepared CSVs from `data/processed/`. The raw archive and interim files are retained for provenance; this project folder does not currently include a one-command pipeline that rebuilds all processed tables from the raw archive.

## 3. Completion checklist

### Already prepared

- [x] Research and document the assignment questions, public dataset choice, and its fit/limitations in `README.md`.
- [x] Preserve the source archive and record its inventory and source notes.
- [x] Prepare the analysis tables, field dictionary, cleaning log, and data-quality report.
- [x] Create and run the exploratory notebook; prepare chart images, workbook, and presentation.
- [x] Keep limitations visible: this is a documented public-data fallback, not the five instructor CSVs; the selected data does not supply test coverage or calendar release dates.

### Do before final assessment submission

- [ ] Ask the instructor/course portal for the five expected CSVs (`modules.csv`, `metrics.csv`, `defects.csv`, `commits.csv`, `releases.csv`) and any quiz/score sheet. The assessment PDF describes the files but does not include their data.
- [ ] If the instructor files are available, check their columns against the assignment, replace the fallback as required, rerun the analysis, and update the workbook, charts, slides, and conclusions. Keep the fallback only if the course permits it.
- [ ] Confirm how the assessment expects the quiz/score sheet to be completed; that sheet is not included in this project.
- [ ] Add native Excel PivotTables/PivotCharts if they are explicitly required, using the prepared analysis table as the source.
- [ ] Verify every assignment question has a directly stated answer, evidence, and a practical recommendation in the final submission.
- [ ] Recheck file names, citations, chart labels, and conclusions against the final dataset, then save the final submission copies.

## 4. Interpretation checks while you work

- The analysis row is intended to represent one source file/module in one release. Check that joins and summaries preserve this grain.
- The churn field is signed net code change (lines added minus lines deleted); a negative value is valid. Use the separate change-volume proxy (added plus deleted lines) when discussing total activity.
- Correlation describes association in this dataset and does not establish that a metric causes defects.
- The public-data fallback is useful for practicing module/release defect analysis, but it cannot answer questions that require missing test-coverage or release-date fields. Do not claim those questions are fully answered by this dataset.
