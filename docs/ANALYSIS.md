# Analysis Guide

This guide describes the preprocessing, evaluation settings, and input requirements for PUMPSERS. The manuscript was submitted to *Advanced Science*.

The data and executable files referenced here are part of the analysis package; they are pending upload to this GitHub repository.

## Preprocessing and Ensemble Weighting

The ensemble combines complementary preprocessing pipelines:

| Constituent | Preprocessing |
|---|---|
| SVM-RBF | ALS → SNV → PCA retaining 97% of variance |
| Random Forest | ALS → MSC → standard scaling |
| XGBoost | Savitzky–Golay filtering → SNV → standard scaling |
| LightGBM | ALS → detrending → L2 normalization → standard scaling |

The current ensemble uses the weighting threshold **τ = 0.20**:

$$
w_i = \frac{\max(a_i - 0.20, 0)}{\sum_{j=1}^{4}\max(a_j - 0.20, 0)},
$$

where $a_i$ is the constituent's inner cross-validation accuracy. If all unnormalized weights are zero, the code uses equal weights. SVM probabilities use `SVC(probability=True)`; tree models use their native `predict_proba()` outputs.

Full-dataset evaluation uses **spectrum-level stratified five-fold cross-validation**, with split seed **999**. Saved memberships preserve the input order and evaluation folds.

## Requirements

The recorded reference runs used **Windows and Python 3.14.0**. Package versions are pinned in [requirements.txt](../requirements.txt), including:

```text
numpy==2.3.4
scipy==1.16.2
scikit-learn==1.7.2
xgboost==3.1.1
lightgbm==4.6.0
pandas==2.3.3
openpyxl==3.1.5
joblib==1.5.2
threadpoolctl==3.6.0
matplotlib==3.10.7
shap==0.50.0
ipykernel>=6
```

From the repository root, install dependencies in the Python environment used by Jupyter:

```bash
python -m pip install -r requirements.txt
```

Restart the Jupyter kernel after installing dependencies. Use an existing Jupyter installation to open the notebooks.

## Usage

### Current Dataset: Retraining and Result Comparison

For the augmented data in this snapshot, start with:

```text
notebooks/06_current_data_reproduction.ipynb
```

1. Keep `code/`, `data/`, `splits/`, and `results/` together in the extracted repository.
2. Open the notebook and confirm the project path printed by the first cell.
3. Choose the analyses in the notebook's configuration cell.
4. Select **Restart Kernel → Run All**.

Default configuration:

```python
RUN_CURATION = True
RUN_FULL = True
RUN_RAW = False
RUN_EXTERNAL = False
RUN_FEWSHOT = False
```

This checks the current inputs, regenerates the curated subset, and trains the full-dataset classifiers using the existing analysis code. Fold accuracies, means, and standard deviations are compared with the archived reference values. Both standard-deviation conventions, `ddof=0` and `ddof=1`, are reported.

New results are written to:

```text
outputs/current_data_comparison/<timestamp>/
├── input_check/
├── curation/
└── full_cv/
    ├── reproduction_comparison.xlsx
    ├── reference_comparison.csv
    ├── accuracy_summary_comparison.csv
    └── verification.json
```

The Excel workbook contains the summary, fold comparison, newly calculated results, and current input identities. Any differences from the reference are retained in the reports.

### External Evaluation and Few-Shot Adaptation

Set `RUN_EXTERNAL = True` for zero-shot evaluation, or `RUN_FEWSHOT = True` for the complete zero-shot and few-shot analysis. The few-shot workflow already includes the zero-shot run.

| Condition | Labeled external spectra added per analyte | Total training spectra | Evaluation spectra | Assignments |
|---|---:|---:|---:|---:|
| 0-shot | 0 | 555 | 100 | 1 |
| 3-shot | 3 | 570 | 85 | 3 |
| 5-shot | 5 | 580 | 75 | 3 |
| 10-shot | 10 | 605 | 50 | 3 |

Each assignment uses identical support and query samples for all classifiers. Query spectra are excluded from fitting. The models are refitted using the training spectra and labeled support samples; model seed 999 is used for these external-evaluation fits. Few-shot uncertainty is the sample standard deviation across three support/query assignments.

`N_JOBS = 16` controls parallelism for external evaluation and adaptation. Reduce it if memory is limited. New predictions and probabilities are compared with the archived reference outputs.

Few-shot results are saved under `outputs/current_data_comparison/<timestamp>/fewshot/`; zero-shot-only results are saved under `.../<timestamp>/external/`. In either directory, `Summary.csv` contains the main performance summary, and `reference_comparison.csv` records prediction differences.

### Original-Input Workflows

Notebooks `00`–`05` and `code/run.py` retain the original-measurement workflow, including strict input checks. They require the original files matching the archived checksums. For the augmented snapshot, use notebook `06` as described above.

The original-input workflows also provide the τ = 0.20 versus 0.50 comparison, raw-spectrum classification, and SHAP analysis. Available command-line tasks can be inspected with:

```bash
python code/run.py --help
```

## Reproducibility Status

- Current input counts, input order, fixed evaluation memberships, and the regenerated 250-spectrum selection have been checked. The current curated membership matches the archived selection.
- **Classification equivalence for the augmented inputs requires fresh training.** Passing input checks or recalculating saved predictions does not establish a new performance result.
- Historical classification scores for the curated 250-spectrum subset remain unresolved. The supported curated workflow covers selection and membership comparison.
- Historical raw-classification and ensemble-SHAP references used τ = 0.50. Their equivalence after changing τ to 0.20 requires separate verification.

[Current-data validation](CURRENT_DATA_CHECK_20261001.json) describes this snapshot. [Reproduction notes](REPRODUCIBILITY.md) and the original packaging audits describe the archived original-input workflow. Acquisition information is provided in [data/metadata/](../data/metadata/); unavailable identifiers are left unspecified.


## Data Identity

The prepared package contains 555 training entries and 100 external spectra. Of the training entries, 80 are noise-augmented replacements of original spectra and 475 retain their reference contents. All 100 external spectra retain their reference contents. The directory name `data/raw/` is retained for loader compatibility; the augmented entries are derived data.

The original measurement checksums are retained in `data/metadata/raw_integrity.csv`. The current-input workflow records the files used in each run and their differences from the archived inputs. Augmented-input results must be evaluated through fresh training rather than inferred from archived scores.
