# Intermediate data

The extracted release-level CSV and JSON files are intentionally excluded from Git because together they are several hundred megabytes. They were used to prepare the smaller analysis tables in `data/processed/`.

The current notebook reads the processed tables directly. The steps used to create the prepared project tables and the source field mappings are documented in `data/processed/cleaning_log.csv`, `data/processed/data_dictionary.csv`, and the project README.
