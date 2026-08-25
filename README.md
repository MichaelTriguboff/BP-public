# WRDS Extraction and Bankruptcy Modelling Pipeline

`BP_Aug_30_Submission.ipynb` builds a monthly, point-in-time bankruptcy-prediction panel for U.S. listed (and later delisted) firms from Compustat FUNDA, CRSP-Compustat (CCM), CRSP monthly stock/index files, Mergent FISD bankruptcy filings, IBES analyst forecasts and short-interest data, then trains and evaluates Random Forest, XGBoost, LightGBM, SVM and Neural Network models against various data sets.

The notebook has **two run modes**:

1. **Local CSV mode (`USE_WRDS = False`; no WRDS account needed)** — uses a pre-built, feature-engineered dataset, normally `wrds_bankruptcy_engineered.csv`. It does **not** require the original Compustat, CCM or CRSP extracts and does **not** delete the local engineered dataset.
2. **WRDS mode (`USE_WRDS = True`)** — connects live to Wharton Research Data Services and rebuilds the panel from source data. A fresh WRDS run deliberately removes prior generated pipeline output before extraction begins, then independently verifies the cleanup in Pipeline Step 2B. This prevents stale files from an earlier run being mistaken for current results.

---

## Public and private repository structure

The project is intended to support both a complete restricted research environment and a curated public reproducibility repository.

| Capability / Content | Private repository | Public repository |
|---|---:|---:|
| Complete notebook | ✓ | ✓ |
| Full methodology and pipeline code | ✓ | ✓ |
| `requirements.txt` | ✓ | ✓ |
| Run with own WRDS access | ✓ | ✓ |
| Run from an authorised engineered CSV | ✓ | ✓* |
| `wrds_bankruptcy_engineered.csv` included | ✓, subject to licence/access restrictions | **No** |
| Raw WRDS extracts | Restricted | **No** |
| Complete working results tree | ✓ | **No** |
| Logs and intermediate files | ✓ | **No** |
| Validation/audit outputs | ✓ | Selected only if appropriate |
| Selected final tables | ✓ | ✓ |
| Selected final figures | ✓ | ✓ |
| WRDS password or other credentials | **Never commit** | **Never commit** |
| `.pgpass` / credential files | **Never commit** | **Never commit** |
| Intended purpose | Full research/reproduction environment | Public code, methodology and selected results |

`*` The user must separately possess an authorised copy of the engineered dataset.

> **Data-access note:** The public repository intentionally excludes licensed WRDS-derived datasets and raw source extracts. Their absence is a licensing and data-governance decision, not an omission from the reproducibility package. Users with appropriate WRDS access can rebuild the data using `USE_WRDS = True`; authorised users who already possess the engineered dataset can run the local pathway using `USE_WRDS = False`.

### Repository architecture

```text
                         PROJECT REPOSITORIES
                                |
                  +-------------+-------------+
                  |                           |
          PRIVATE REPOSITORY           PUBLIC REPOSITORY
                  |                           |
        Full research record        Reproducibility package
                  |                           |
        Notebook                    Notebook
        README                      README
        requirements.txt            requirements.txt
        authorised engineered data  .gitignore
        authorised raw data         data/README.md
        complete results            selected results/
        validation outputs          selected figures/tables
        logs                        no licensed data
                  |                           |
          RESTRICTED ACCESS              PUBLIC ACCESS
```

The private repository is the working research environment. Access to licensed data should be limited to authorised users. The public repository is deliberately curated so that source code and methodology can be inspected without redistributing restricted WRDS-derived data.

## Choose the operating mode before running

### A. No WRDS access — local reproduction

Use this route if you cannot connect to WRDS or do not have an institutional WRDS subscription.

1. Obtain the supplied feature-engineered file `wrds_bankruptcy_engineered.csv`.
2. Put it in:
   ```
   <BP_PROJECT_ROOT>/data/final/wrds_bankruptcy_engineered.csv
   ```
3. Set the `BP_PROJECT_ROOT` environment variable to the `Final` project/output directory.
4. Start Jupyter or VS Code and open the notebook.
5. In **Pipeline Step 2A**, set:
   ```python
   USE_WRDS = False
   ```
6. Run Step 2A. In local mode the cleanup routine is deliberately skipped, so the supplied engineered dataset is preserved.
7. Run **Pipeline Step 2B**. It checks that a supported local modelling dataset exists. It does **not** require `FUNDA_FILE`, `CCM_FILE` or `CRSP_FILE`.
8. Confirm that Step 2B ends with:
   ```
   READY TO RUN USING LOCAL DATA
   Pre-flight OK — safe to run the rest of the pipeline.
   ```
9. Run the remaining notebook cells from top to bottom.

The preferred local input is `wrds_bankruptcy_engineered.csv`. Supported alternatives are `final_dataset.csv`, a configured `MODEL_FILE`, or all three split files (`train_dataset.csv`, `validation_dataset.csv`, and `test_dataset.csv`).

### B. WRDS-connected fresh reproduction

Use this route only when the required WRDS subscriptions and credentials are available.

1. **Restart the kernel** before starting the full run.
2. In **Pipeline Step 2A**, set:
   ```python
   USE_WRDS = True
   WRDS_USERNAME = "<your_wrds_username>"
   ```
3. Run Step 2A **before any extraction or modelling cells**.
4. Step 2A removes prior generated material from the pipeline output locations, including the main results tree, prior WRDS raw extracts, generated final datasets, logs and validation artefacts. It also checks legacy `results/` and `validation/` locations relative to the notebook working directory.
5. Step 2A recreates the empty directory structure required by later cells.
6. Run **Pipeline Step 2B**.
7. Step 2B is an independent safeguard: it scans the generated-output locations and stops with an `AssertionError` if stale files remain.
8. Do not continue unless Step 2B reports:
   ```
   CLEANUP VERIFIED
   READY FOR FRESH WRDS EXTRACTION
   Pre-flight OK — safe to run the rest of the pipeline.
   ```
9. Continue through the notebook sequentially. When the WRDS connection is opened, enter the required WRDS credentials if prompted.

If Step 2B reports `WRDS PRE-FLIGHT FAILED`, **do not continue the extraction**. Review the listed files, restart the kernel, rerun Steps 1, 2A and 2B, and resolve any file that could not be removed.

### Why the WRDS cleanup is performed

A WRDS run is intended to be a fresh end-to-end reconstruction. Removing previous generated output before extraction prevents old CSVs, plots, model tables, validation artefacts or intermediate WRDS extracts from being confused with files created by the current run. Step 2A performs the cleanup; Step 2B independently verifies that the cleanup succeeded.

## Quickstart (terminal)

```bash
# 1. Create a project folder and move into it
mkdir bankruptcy-pipeline
cd bankruptcy-pipeline

# 2. Put the notebook, README.md, requirements.txt, and your data folder here, e.g.:
#    bankruptcy-pipeline/
#      BP_Aug_30_Submission.ipynb
#      README.md
#      requirements.txt
#      Final/data/final/wrds_bankruptcy_engineered.csv

# 3. Create and activate a virtual environment
python3 -m venv venv
source venv/bin/activate          # macOS/Linux
# venv\Scripts\activate           # Windows PowerShell

# 4. Upgrade pip, then install dependencies
pip install --upgrade pip
pip install -r requirements.txt

# 5. (Optional) point output/data to a custom location instead of ./Final
export BP_PROJECT_ROOT="$(pwd)/Final"     # macOS/Linux
# setx BP_PROJECT_ROOT "C:\path\to\Final" # Windows (restart terminal after)

# 6. Launch Jupyter
jupyter lab
# or: jupyter notebook
```

Then in the browser tab that opens:
1. Open `BP_Aug_30_Submission.ipynb`
2. In the Step 2A config cell, set `USE_WRDS = False` (unless you have your own WRDS credentials — then set your `WRDS_USERNAME` instead)
3. Run all cells: **Run → Run All Cells** (or **Kernel → Restart Kernel and Run All Cells**)

### Non-interactive execution (no browser UI)

To run the whole notebook end-to-end from the terminal and save an executed copy:

```bash
jupyter nbconvert --to notebook --execute BP_Aug_30_Submission.ipynb --output BP_Aug_30_Submission_executed.ipynb
```

The sections below cover requirements, file placement, and configuration in more detail.

---



## 1. Requirements

- **Python 3.10** (the notebook was built/pinned against 3.10.11)
- **Jupyter** (JupyterLab, Jupyter Notebook, or VS Code's notebook support)
- ~4–8 GB RAM minimum for the local-CSV path; more (and a fast disk) if running in WRDS mode, since CRSP/Compustat pulls are large
- Internet access only if running in WRDS mode

### Python packages

A `requirements.txt` is included alongside this README. Install everything with:

```bash
pip install -r requirements.txt
```

Or install packages individually:

```bash
pip install pandas numpy scikit-learn xgboost lightgbm tensorflow \
            matplotlib openpyxl python-docx wrds jupyterlab
```

| Package | Used for |
|---|---|
| pandas, numpy | data wrangling |
| scikit-learn | Random Forest, SVM, logistic regression, imputation, scaling, calibration, metrics |
| xgboost | gradient boosting model |
| lightgbm | gradient boosting benchmark (Step 46E — notebook skips this step gracefully if not installed) |
| tensorflow | the Keras Neural Network models |
| matplotlib | all plots (class imbalance, calibration curves, feature importance, etc.) |
| openpyxl | writing formatted `.xlsx` comparison tables |
| python-docx (`docx`) | writing formatted `.docx` comparison tables |
| wrds | only needed for WRDS mode — connecting to the WRDS PostgreSQL server. |
| jupyterlab | notebook environment to open/run the `.ipynb` file |

If you don't need WRDS mode, you can remove/skip the `wrds` line in `requirements.txt`.

---

## 2. Files needed to run in local mode

Place these in the notebook's data folder (see "Output/data locations" below — default is `./Final/data/final/`, but confirm against whatever `FINAL_DIR` resolves to when you run the config cell):

- `wrds_bankruptcy_engineered.csv` — the full merged, feature-engineered panel (`model_df`). This is the primary file the notebook looks for.
- Optional fallbacks the loader will also check, in order: `final_dataset.csv`, the file pointed to by `MODEL_FILE`, `train_dataset.csv`, `validation_dataset.csv`, `test_dataset.csv`.

If none of these files are found, the notebook raises a `FileNotFoundError` naming exactly what it expected — the config cell prints the resolved path it will look in.

---

## 3. Setup steps

1. Copy the notebook, README, requirements file and (for local mode) the supplied engineered CSV into the project structure, for example:
   ```
   project/
     BP_Aug_30_Submission.ipynb
     README.md
     requirements.txt
     Final/
       data/
         final/
           wrds_bankruptcy_engineered.csv
   ```
2. Create and activate a virtual environment, then install the packages from `requirements.txt`.
3. Set `BP_PROJECT_ROOT` to the `Final` directory.
4. Launch Jupyter from the project directory:
   ```bash
   jupyter lab
   # or
   jupyter notebook
   ```
5. Open `BP_Aug_30_Submission.ipynb`.
6. Run Pipeline Step 1.
7. In **Step 2A**, choose the operating mode:
   ```python
   USE_WRDS = False   # local engineered-data reproduction
   # OR
   USE_WRDS = True    # fresh WRDS extraction
   ```
8. Run Step 2A, then Step 2B. Read the printed pre-flight output before continuing.
9. Only after Step 2B reports the appropriate `READY` message should the remaining cells be run from top to bottom.
---

## 4. Running in WRDS mode (requires WRDS access)

A live run requires an institutional WRDS subscription covering the source databases used by the notebook.

### Fresh-run procedure

1. Restart the kernel.
2. Set `USE_WRDS = True` and set `WRDS_USERNAME` to your own WRDS username in Step 2A.
3. Run Step 2A. The notebook deletes prior generated pipeline output and recreates the required folders.
4. Run Step 2B. This independently verifies that the generated-output locations contain no stale files.
5. Continue only when `CLEANUP VERIFIED` and `READY FOR FRESH WRDS EXTRACTION` are displayed.
6. When `wrds.Connection()` runs, supply your WRDS credentials if prompted.
7. Run all remaining cells sequentially.
8. The notebook closes the WRDS connection at the end of the pipeline.

The full extraction includes Compustat FUNDA, CCM, CRSP monthly stock/index data, FISD bankruptcy information, IBES data and the short-interest extension. It can be compute-, network- and time-intensive (with the full extraction taking approximately three hours in this study) depending on WRDS server load.

### Cleanup safeguard

The cleanup is intentionally restricted to `USE_WRDS = True`. Local mode must retain the engineered CSV used as its input. In WRDS mode, Step 2A removes old generated files; Step 2B then verifies the cleaned state. If Step 2B detects any remaining file in a generated-output location, it stops execution rather than allowing a potentially contaminated run.

---

## 5. Configuring output location

By default, all results and intermediate files are written to `./Final` relative to wherever you launch Jupyter. To redirect this (e.g. to a shared drive or a consistent path across machines), set the environment variable **before** starting Jupyter:

```bash
# macOS/Linux
export BP_PROJECT_ROOT="/path/to/output/Final"
jupyter lab

# Windows PowerShell (persists across sessions)
setx BP_PROJECT_ROOT "C:\Users\<you>\Dev\Final"
# then restart your terminal / Jupyter
```

Sub-folders created automatically under `PROJECT_ROOT`:
- `results/` — model comparison tables, plots
- `results/feature_importance/`
- `results/short_interest_extension/`
- `results/ibes/`
- `results/baseline/per_feature_set/` — Word (`.docx`) and Excel (`.xlsx`) comparison tables
- `results/calibration/` — reliability curve plots
- `validation/` — look-ahead-bias audit sample (`lookahead_violations_sample.csv`)
- `data/final/` — where the engineered CSV / train/validation/test CSVs are expected/saved

---

## 6. What the notebook produces

Running the full notebook (256 cells) produces, among other things:
- A cleaned, point-in-time, forward-looking (12-month) bankruptcy hazard panel with a combined failure target (`distress_event_any`) drawing on CRSP delisting codes and Mergent FISD Chapter 11/7 filings
- Three engineered feature sets: Barboza/Appendix, Shumway-style, and Combined
- Trained Random Forest, XGBoost, LightGBM, SVM (optional), and Neural Network models per feature set, plus Altman (1968) and refit-logit baselines
- ROC-AUC, PR-AUC, recall, precision, top-decile capture metrics with paired bootstrap confidence intervals
- Isotonic probability calibration and Brier-score/reliability-curve diagnostics
- IBES analyst-forecast and short-interest extension analyses
- Cost-sensitive threshold and permutation-importance analyses
- Formatted `.docx`/`.xlsx` summary tables and `.csv`/plot outputs throughout


