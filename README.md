# Plant-Inspired Nanoribbon Pump for Transfer-Free Microvolume SERS with Weighted Ensemble Learning

**PUMPSERS**

> Repository setup in progress: this initial commit contains the README. The code, notebooks, data, license, and reference files described below will be uploaded separately. File links and execution instructions become usable once those files are present.

Data and analysis code accompanying the manuscript on nanoribbon-based microvolume surface-enhanced Raman scattering (SERS) and weighted ensemble classification.

The repository provides spectral preprocessing, baseline curation, classification, SHAP analysis, and supplementary zero-shot and few-shot evaluation.

## Authors

Seokju Choi<sup>1,†</sup>, Sunhee Park<sup>2,†</sup>, Dodam Kim<sup>2,†</sup>, Minki Kim<sup>1</sup>, Sangsoon Oh<sup>3</sup>, Junseong Ahn<sup>1,2,&#42;</sup>, Sunae So<sup>1,2,&#42;</sup>

1. Department of Control and Instrumentation Engineering, Korea University, Sejong 30019, Republic of Korea
2. Department of Electro-Mechanical System Engineering, Korea University, Sejong 30019, Republic of Korea
3. School of Physics and Astronomy, Cardiff University, Cardiff CF24 3AA, United Kingdom

† These authors contributed equally.  
\* Corresponding authors.

## Repository Structure

```text
PUMPSERS/
├── README.md
├── START_HERE_KO.md
├── LICENSE
├── requirements.txt
├── code/
│   ├── config.py                      # Parameters and paths
│   ├── data_loader.py                 # Manifest-based loading
│   ├── interpolation.py               # Common wavenumber grid
│   ├── preprocessing.py               # Spectral transformations
│   ├── baseline_curation.py           # Baseline-based spectrum selection
│   ├── classification.py              # Classification and five-fold CV
│   ├── external_test.py               # Model fitting and prediction helpers
│   ├── new_substrate_fewshot.py        # External evaluation and adaptation
│   ├── weight_baseline_comparison.py   # Weighting threshold comparison
│   ├── shap_analysis.py               # SHAP and reference-peak analysis
│   ├── visualization.py               # Figures and tables
│   ├── utils.py                       # Ensemble weighting and metrics
│   ├── evaluation.py                  # Original-input validation and evaluation
│   ├── check_package.py               # Original-package integrity audit
│   ├── current_data_check.py           # Current-input retraining and comparison
│   └── run.py                         # Original-input command-line entry point
├── notebooks/
│   ├── 00_verify_data.ipynb
│   ├── 01_full_dataset.ipynb
│   ├── 02_external_test.ipynb
│   ├── 03_new_substrate_fewshot.ipynb
│   ├── 04_interpretability.ipynb
│   ├── 05_weighting_sensitivity.ipynb
│   └── 06_current_data_reproduction.ipynb
├── data/
│   ├── raw/                          # 555 training entries; see data status below
│   │   ├── Chlorpyrifos/
│   │   ├── Fipronil/
│   │   ├── Indole/
│   │   ├── Thiabendazole/
│   │   └── Thiram/
│   ├── external_test/                # 100 supplementary test spectra
│   ├── metadata/                     # Acquisition and file-identity records
│   ├── manifest.csv                  # Training order and curated membership
│   └── manifest_external.csv         # External input order
├── splits/
│   ├── full_cv.csv                   # Five-fold training/test memberships
│   ├── curated_membership.csv
│   └── fewshot_assignments.csv        # Support/query assignments
├── results/
│   └── reference/                    # Archived comparison results
│       ├── full_cv/
│       ├── raw_cv/
│       └── external/
├── docs/
│   ├── DATA_DICTIONARY.md
│   ├── REPRODUCIBILITY.md
│   ├── CURRENT_DATA_CHECK_KO.md
│   ├── reference_peaks.csv
│   └── provenance/                   # Source and input-history records
└── outputs/                          # Created when analyses are run
```

## Data Description

The original collection comprises 555 spectra from five analytes. The supplementary test cohort contains 100 spectra, with 20 spectra per analyte. Spectra are loaded onto a common grid of **918 points spanning 500–1700 cm⁻¹**.

| Analyte | Training entries | External test spectra | Curated subset target |
|---|---:|---:|---:|
| Chlorpyrifos | 124 | 20 | 50 |
| Fipronil | 105 | 20 | 50 |
| Indole | 202 | 20 | 50 |
| Thiabendazole | 65 | 20 | 50 |
| Thiram | 59 | 20 | 50 |
| **Total** | **555** | **100** | **250** |

**Current data status.** This evaluation snapshot includes **80 noise-augmented training entries** in place of their original versions; the other 475 training entries and all 100 external spectra retain their reference contents. The directory name `data/raw/` is retained for loader compatibility. The augmented entries are derived data. The training total remains 555.

Original measurement checksums remain in `data/metadata/raw_integrity.csv`. The current-input workflow records the files actually used and their differences from the archived inputs. See [current-data checks](docs/CURRENT_DATA_CHECK_KO.md) and [input-preservation records](docs/provenance/current_data_preservation.json).

Baseline curation selects **50 spectra per analyte** using baseline level and spectral slope with K-means clustering and distance-based selection. The 250-spectrum subset is generated by the code.

## Classification Models

Three main classifiers are compared on identical evaluation samples:

| Model | Method |
|---|---|
| SVM-Linear | Linear SVM with ALS baseline correction, SNV normalization, and PCA |
| SVM-RBF | RBF-kernel SVM with the same ALS–SNV–PCA sequence |
| Weighted Ensemble | Weighted probability voting over SVM-RBF, Random Forest, XGBoost, and LightGBM |

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

The recorded reference runs used **Windows and Python 3.14.0**. Package versions are pinned in [requirements.txt](requirements.txt), including:

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

[Current-data validation](docs/CURRENT_DATA_CHECK_20261001.json) describes this snapshot. [Reproduction notes](docs/REPRODUCIBILITY.md) and the original packaging audits describe the archived original-input workflow. Acquisition information is provided in [data/metadata/](data/metadata/); unavailable identifiers are left unspecified.

## Citation

If you use this work, please cite the manuscript:

```text
S. Choi, S. Park, D. Kim, M. Kim, S. Oh, J. Ahn, and S. So,
"Plant-Inspired Nanoribbon Pump for Transfer-Free Microvolume SERS
with Weighted Ensemble Learning."
Manuscript.
```

The final journal citation and DOI will be added when available.

## License

The code is distributed under the MIT License. See [LICENSE](LICENSE).

## Contact

- **Sunae So:** [sunaeso@korea.ac.kr](mailto:sunaeso@korea.ac.kr)
- **Junseong Ahn:** [junseong@korea.ac.kr](mailto:junseong@korea.ac.kr)
