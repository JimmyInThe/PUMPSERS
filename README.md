# Plant-Inspired Nanoribbon Pump for Transfer-Free Microvolume SERS with Weighted Ensemble Learning

**PUMPSERS** — Data and analysis code for the manuscript submitted to *Advanced Science*.

This project combines a plant-inspired nanoribbon pump for microvolume surface-enhanced Raman scattering (SERS) with machine-learning classification. The analysis package covers spectral preprocessing, baseline curation, classification, SHAP interpretation, and external evaluation.

## Authors

Seokju Choi<sup>1,†</sup>, Sunhee Park<sup>2,†</sup>, Dodam Kim<sup>2,†</sup>, Minki Kim<sup>1</sup>, Sangsoon Oh<sup>3</sup>, Junseong Ahn<sup>1,2,&#42;</sup>, Sunae So<sup>1,2,&#42;</sup>

1. Department of Control and Instrumentation Engineering, Korea University, Sejong 30019, Republic of Korea
2. Department of Electro-Mechanical System Engineering, Korea University, Sejong 30019, Republic of Korea
3. School of Physics and Astronomy, Cardiff University, Cardiff CF24 3AA, United Kingdom

† These authors contributed equally.  
\* Corresponding authors.

## Data

The analysis uses five analytes: chlorpyrifos, fipronil, indole, thiabendazole, and thiram. Each spectrum is interpolated onto **918 wavenumber points from 500 to 1700 cm⁻¹**.

| Dataset | Spectra | Description |
|---|---:|---|
| Training dataset | 555 | Five analytes; counts of 124, 105, 202, 65, and 59, respectively |
| Baseline-curated subset | 250 | 50 spectra per analyte, selected from the training dataset |
| External test dataset | 100 | 20 spectra per analyte |

The prepared training package includes **80 noise-augmented entries** in place of their original versions. These are derived spectra; input identities and processing details are documented in the package metadata and [analysis guide](docs/ANALYSIS.md#data-identity).

## Package Structure

```text
PUMPSERS/
├── code/                 # Preprocessing, classification, and interpretation
├── notebooks/            # Jupyter analysis workflows
├── data/
│   ├── raw/              # Training spectra, including the derived entries noted above
│   ├── external_test/    # External test spectra
│   └── metadata/         # Acquisition information and file identities
├── splits/               # Cross-validation and few-shot assignments
├── results/reference/   # Reference results
├── docs/                 # Analysis settings and dataset documentation
├── requirements.txt
├── LICENSE
└── README.md
```

This describes the complete analysis package. The GitHub repository currently contains documentation; data and executable files are pending upload.

## Analysis

- **Preprocessing and curation:** spectral interpolation, baseline correction, normalization, and baseline-based spectrum selection.
- **Classification:** linear SVM, RBF SVM, and a weighted ensemble combining RBF SVM, Random Forest, XGBoost, and LightGBM.
- **Interpretation:** feature importance and SHAP analysis.
- **External evaluation:** zero-shot classification and few-shot adaptation using labeled spectra from the external dataset.

The full-dataset classifiers use spectrum-level stratified five-fold cross-validation with seed **999**. Ensemble weights are calculated from inner cross-validation accuracy using a threshold of **0.20**. Detailed settings and evaluation protocols are provided in the [analysis guide](docs/ANALYSIS.md).

## Installation

The recorded reference environment used Windows with Python **3.14.0**. Dependency versions are specified in `requirements.txt`.

Once the complete package is available, install dependencies from its root directory:

```bash
python -m pip install -r requirements.txt
```

Use the same Python environment for Jupyter and restart the kernel after installation.

## Usage

For the prepared dataset, open:

```text
notebooks/06_current_data_reproduction.ipynb
```

1. Keep the package folders together and confirm the project path in the first cell.
2. Select the analyses in the notebook's configuration cell.
3. Run **Restart Kernel → Run All**.

The default configuration performs baseline curation and full-dataset classification. External evaluation and few-shot adaptation can be enabled in the same notebook. Results are saved under `outputs/current_data_comparison/`.

The curated workflow covers subset selection. Other notebooks include interpretation and weighting analyses with their stated input requirements. See the [analysis guide](docs/ANALYSIS.md) for notebook selection, output files, and reference-result comparisons.

## Citation

```text
S. Choi, S. Park, D. Kim, M. Kim, S. Oh, J. Ahn, and S. So,
"Plant-Inspired Nanoribbon Pump for Transfer-Free Microvolume SERS
with Weighted Ensemble Learning."
Manuscript submitted to Advanced Science.
```

The final bibliographic details and DOI will be added when available.

## License

The analysis code uses the MIT License. See the `LICENSE` file accompanying the code package.

## Contact

- **Sunae So:** [sunaeso@korea.ac.kr](mailto:sunaeso@korea.ac.kr)
- **Junseong Ahn:** [junseong@korea.ac.kr](mailto:junseong@korea.ac.kr)
