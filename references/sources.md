# Data source and research record

## Selected dataset

- **Name:** SmartSHARK release-level defect prediction data, from the replication package for “Problems with SZZ and Features: An empirical study of the state of practice of defect prediction data collection.”
- **Repository record:** https://zenodo.org/records/5675024
- **DOI:** 10.5281/zenodo.5675024
- **Selected file:** `release-level-data.tar.gz` (do not use the separately named `DO_NOT_USE_release-level-data.tar.gz`).
- **Published size:** 616,057,233 bytes.
- **Published MD5:** `e81b745db794beecfcabd4e8cff14036`.
- **License:** Apache 2.0 according to the replication record.
- **Scope stated by the authors:** labeled release-level data for 398 releases. The study describes 38 Apache projects and file-level software metrics, change/churn features, defect counts, and issue metadata including severity.
- **Retrieval date:** 2026-10-13 (local date).
- **Local archive path:** `data/raw/release-level-data.tar.gz`.

## Known fit limitations

- The assignment asks for test coverage; no coverage field was found in the selected Calcite aggregate files.
- `MOSER_authors` is the distinct-author count and `MOSER_revisions` is the file revision count. Treat these as file-level process measures, not a separate commit table.
- `MOSER_sum_code_churn` is signed net change (sum of lines added minus lines deleted); negative values are valid. The analysis uses a separately named change-volume proxy equal to summed lines added plus summed lines deleted.
- The source is file-level; this project uses file/class as the module proxy and retains project/release identifiers.
- Calendar release dates are not present for this selected subset; charts use semantic release order.
- The archive is prepared research data, not the five assignment CSVs. Field mappings and adaptations are recorded in `../data/processed/data_dictionary.csv` and `../data/processed/cleaning_log.csv`.

## Implemented subset and audit

- Project subset: Apache Calcite, 16 releases from 1.0.0 through 1.15.0.
- Analysis grain: one Java source file in one release.
- Rows: 19,751 module-release records, covering 1,644 distinct Java files and 24 package components.
- Defect labels: 2,138. Severity issue identifiers deduplicated per file-release reconcile to the same total.
- Duplicate module-release keys: 0. Unmatched severity JSON file paths: 0.
- Test coverage: unavailable. Release dates: unavailable.
- Negative signed net-change records: 1,253; preserved because they represent net line deletions. Negative change-volume proxy records: 0.

## Metric definition source

- SmartSHARK's implementation of the MOSER change metrics defines signed code churn as summed lines added minus summed lines deleted: https://smartshark.github.io/mynbou/_modules/metrics/change.html.
- Lee et al., “Micro Interaction Metrics for Defect Prediction” (2011), lists revisions, authors, LOC added/deleted, and CodeChurn as change-history metrics: https://doi.org/10.1145/2025113.2025156.

## Research sources

- Herbold et al. study: https://link.springer.com/article/10.1007/s10664-021-10092-4
- SmartSHARK data-release documentation: https://smartshark.github.io/dbreleases/
- Assessment brief: `CH28.pdf` supplied by the user.
