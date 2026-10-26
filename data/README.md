# Data

No data is included in this repository.

| File (expected name)                        | Content                                              | Used by            |
|---------------------------------------------|------------------------------------------------------|--------------------|
| `002_processed_data/xy_df.csv`              | SMILES, stability label, HOMO/LUMO, RDKit descriptors | notebook 01        |
| `002_processed_data/df_combined_hopv_sql.csv` | HOPV + 232 extra molecules from the course SQL database | notebook 02    |
| `003_kaggle_competition_data/25261_classification_sample_submission.csv` | 200 unlabeled test SMILES | notebooks 01, 03 |
| `003_kaggle_competition_data/25261_regression_sample_submission.csv`     | 200 unlabeled test SMILES | notebooks 02, 04 |

## Sources

- **Harvard Organic Photovoltaic Dataset (HOPV15)**: 350 molecules with experimental/computed properties.
  Lopez et al., *The Harvard organic photovoltaic dataset*, Scientific Data 3, 160086 (2016).
- **Extra molecules and Kaggle test sets**: provided by the Breda University of Applied Sciences course
  (Block 1B Machine Learning, 2025-26) and not redistributed here.

The notebooks read from `..\..\..\data\...` (the original course folder layout). To run them, either recreate that
layout or edit the paths in the first cells.
