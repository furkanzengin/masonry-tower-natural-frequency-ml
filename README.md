
# Masonry Tower Natural Frequency Prediction with Explainable Machine Learning

This repository contains the **datasets and source code** associated with the manuscript:

> **Finite Element-Based Explainable Machine Learning for Predicting the Natural Frequencies of Historical Masonry Towers**

The study develops a finite element-based and explainable machine learning framework for predicting the first two natural frequencies of isolated historical masonry towers. The database was generated from Abaqus modal analyses and the machine learning workflow was designed to avoid geometry-related data leakage by separating tower geometries between training, validation, and test subsets.

This repository intentionally contains **only the datasets and code** required for the analyses. Generated figures, tables, trained models, and other result files are not included in version control.

## Repository Structure

```text
masonry-tower-natural-frequency-ml/
│
├── README.md
├── .gitignore
│
├── data/
│   ├── masonry_tower_primary_dataset.xlsx
│   └── masonry_tower_geometry_reference_dataset.xlsx
│
└── code/
    ├── 01_data_validation.ipynb
    ├── 02_exploratory_data_analysis.ipynb
    ├── 03_data_preparation.ipynb
    ├── 04_baseline_model_evaluation.ipynb
    ├── 05_hyperparameter_optimization.ipynb
    ├── 06_optimized_model_reproducibility.ipynb
    └── 07_model_selection_and_shap.ipynb
```

## Dataset Description

Two datasets are provided.

### 1. Primary dataset

**File:** `data/masonry_tower_primary_dataset.xlsx`

The primary dataset contains **1,146 records** obtained from **191 distinct tower geometries**, with each geometry evaluated using **six different material combinations**. Young's modulus varies between 500 and 6000 MPa, while density varies between 1200 and 2700 kg/m³.

The dataset contains nine input variables and two target variables:

| Column | Description | Role |
|---|---|---|
| `Height (m)` | Tower height | Input |
| `Section a (m)` | Cross-sectional dimension in the x-direction | Input |
| `Section b (m)` | Cross-sectional dimension in the y-direction | Input |
| `Wall Thickness (m)` | Wall thickness | Input |
| `Opening z/H` | Relative vertical position of the opening | Input |
| `Opening Ratio x (%)` | Opening ratio in the x-direction | Input |
| `Opening Ratio y (%)` | Opening ratio in the y-direction | Input |
| `E (MPa)` | Young's modulus | Input |
| `d (kg/m3)` | Material density | Input |
| `f1 (Hz)` | First natural frequency | Target |
| `f2 (Hz)` | Second natural frequency | Target |

For tower configurations without an opening in a given direction, the corresponding opening ratio is represented as zero.

### 2. Geometry reference dataset

**File:** `data/masonry_tower_geometry_reference_dataset.xlsx`

This dataset contains **191 unique tower geometries** and was created to examine the influence of geometric parameters while keeping the material properties constant. In the finite element analyses used to construct this reference dataset, Young's modulus was fixed at **3000 MPa** and density at **1400 kg/m³**.

The geometry reference dataset is **not an independent validation dataset**. It is used as a reference for comparing the influence of geometric and material parameters on natural frequency estimation.

## Finite Element Data Generation

The natural-frequency data were generated using three-dimensional finite element modal analyses in **Abaqus**. The isolated masonry towers were modeled as homogeneous, isotropic, linear-elastic solid bodies. The models used 10-node quadratic tetrahedral solid elements (C3D10) and fixed-base boundary conditions. The first and second natural frequencies obtained from eigenvalue analyses were used as the machine learning targets.

## Machine Learning Workflow

To prevent information leakage, samples derived from the same tower geometry are kept together throughout data splitting and cross-validation.

A `Geometry_ID` is constructed from the seven geometric variables and is used only for grouping; it is not included as a machine learning input feature. The primary dataset is split at the geometry level using `GroupShuffleSplit`:

- Training set: **912 records from 152 geometries**
- Test set: **234 records from 39 geometries**
- Training proportion: **79.58%**
- Test proportion: **20.42%**

Model comparison and hyperparameter optimization use **five-fold `GroupKFold` cross-validation** on the training set so that the same geometry cannot appear in different folds.

The following regression algorithms are evaluated:

- Linear Regression
- Ridge Regression
- K-Nearest Neighbors Regression
- Decision Tree Regression
- Random Forest Regression
- Gradient Boosting Regression
- Support Vector Regression with an RBF kernel

Scale-sensitive models are implemented using `StandardScaler` within scikit-learn pipelines. Hyperparameter optimization is performed for Support Vector Regression and Gradient Boosting Regression using both **Randomized Search** and **Particle Swarm Optimization (PSO)** under an equal computational budget. SHAP analysis is then used to interpret the selected models.

## Notebook Execution Order

The notebooks are intended to be run in numerical order:

| Notebook | Purpose |
|---|---|
| `01_data_validation.ipynb` | Checks dataset structure, descriptive properties, missing values, duplicates, and geometry consistency |
| `02_exploratory_data_analysis.ipynb` | Performs exploratory analysis and generates descriptive plots and correlations |
| `03_data_preparation.ipynb` | Creates `Geometry_ID`, geometry-based train/test assignments, and GroupKFold definitions |
| `04_baseline_model_evaluation.ipynb` | Evaluates the seven baseline regression models |
| `05_hyperparameter_optimization.ipynb` | Performs Randomized Search and PSO optimization for SVR and GBR |
| `06_optimized_model_reproducibility.ipynb` | Reconstructs and checks optimized models and reproducibility-related analyses |
| `07_model_selection_and_shap.ipynb` | Performs final model comparison, model selection, and SHAP explainability analysis |

Running the notebooks sequentially is recommended because later notebooks use intermediate files generated by earlier stages, particularly the geometry-based split assignments and optimization outputs.

## File Paths and Portability

The notebooks do **not** depend on user-specific absolute file paths. The project root and `data/` directory are detected programmatically, so the repository can be placed in any local directory as long as the repository structure is preserved.

For example, the same notebooks can be run from locations such as:

```text
C:/Users/User/Documents/masonry-tower-natural-frequency-ml/
D:/Research/masonry-tower-natural-frequency-ml/
/home/user/projects/masonry-tower-natural-frequency-ml/
```

No modification of dataset paths should be required.

## Generated Outputs

When the notebooks are executed, generated files are written to a local `outputs/` directory. Depending on the notebook, these files may include figures, tables, model files, split assignments, optimization records, and SHAP outputs.

The `outputs/` directory is excluded through `.gitignore`, so generated results are **not stored in this repository**.

## Software Requirements

The notebooks were developed using **Python 3.9.13** with Jupyter and rely on the following main packages:

```text
numpy
pandas
scipy
scikit-learn
matplotlib
seaborn
joblib
shap
openpyxl
jupyter
```

A compatible environment can be prepared with:

```bash
pip install numpy pandas scipy scikit-learn matplotlib seaborn joblib shap openpyxl jupyter
```

## Running the Code

1. Clone or download this repository.
2. Keep the `data/` and `code/` directories in their original locations.
3. Open a terminal in the repository root.
4. Start Jupyter:

```bash
jupyter notebook
```

5. Open the `code/` directory.
6. Run the notebooks sequentially from `01_data_validation.ipynb` to `07_model_selection_and_shap.ipynb`.

The notebooks automatically locate the datasets relative to the repository root.

## Reproducibility

The workflow uses geometry-based grouping to prevent samples belonging to the same tower geometry from appearing in both training and test data or across validation folds. A fixed random seed is used for random processes in the main experimental workflow to support reproducibility.

The PSO implementation is included directly in the notebooks and does not require an external PSO package.

## Citation

If you use the datasets or code provided in this repository, please cite the associated manuscript:

> **Finite Element-Based Explainable Machine Learning for Predicting the Natural Frequencies of Historical Masonry Towers**

Full bibliographic information and DOI can be added here after publication.

## Notes

The datasets are based on numerically generated finite element results. The developed machine learning models are intended for tower configurations within the geometric and material ranges represented in the provided database. The finite element models assume fixed-base conditions and homogeneous linear-elastic masonry behavior.
