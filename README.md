# VGICs-pred

**Homology-controlled protein language model-based annotation of voltage-gated ion channels**

[![Python](https://img.shields.io/badge/Python-3.12-3776AB)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.5.1-EE4C2C)](https://pytorch.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**Repository:** https://github.com/gaozhiw/VGICs-pred  
**Manuscript:** *VGICs-pred: a homology-controlled protein language model method for voltage-gated ion channel annotation and exploratory sequence-feature analysis*  
**Journal:** *Bioinformatics* (accepted; final publication details pending)  
**Archived software version and test data:** **[ADD VERIFIED, VERSION-SPECIFIC ZENODO DOI BEFORE RELEASE]**

> **Publication note:** The Zenodo DOI above must refer to an archive containing the version of the software used for the accepted manuscript and the corresponding test data. Do not replace it with an unrelated or older archive.

## Overview

VGICs-pred is a research workflow for predicting whether a protein sequence is associated with the voltage-gated ion channel (VGIC) class. It combines protein language model representations, particularly ProtT5 embeddings, with a multilayer perceptron (MLP) classifier. The repository also contains code for homology-aware dataset preparation, internal benchmarking, BLAST-based comparisons, candidate sequence annotation, and exploratory sequence-feature analyses.

The study emphasizes evaluation under controlled sequence similarity and transparent reporting of the limits of candidate annotation. Predictions are **computational hypotheses**, not experimental confirmation of ion-channel activity or function.

### Main components

- **Sequence preparation:** FASTA processing, dataset partitioning, and sequence-similarity checks.
- **Protein representations:** ProtT5, with additional ProtBert, ESM2, and CKSAAP analyses in the research notebooks.
- **Classification:** MLP-based binary prediction of VGIC versus non-VGIC sequences.
- **Robustness:** Analyses across five random seeds: `42`, `2023`, `1234`, `567`, and `100`.
- **Comparative evaluation:** BLAST baseline calculations and additional research benchmarking.
- **Exploratory analysis:** Candidate predictions, CKSAAP feature tables, and related analyses.

## Repository structure

The following entries correspond to the current public repository structure. Some output directories/files are produced **after** running notebooks and are not necessarily included in a fresh clone.

```text
VGICs-pred/
├── 01_data_split_and_homology_control.ipynb
├── 02_feature_encoding_training_and_benchmarking.ipynb
├── 03_blast_baseline_comparison.ipynb
├── 04_unreviewed_sequence_prediction.ipynb
├── data_input/
├── split_seed_42/
├── split_seed_2023/
├── split_seed_1234/
├── split_seed_567/
├── split_seed_100/
├── saved_models_2026_eval/
├── CKSAAP_MRMD/
├── GO_VGIC_analysis_github/
├── environment.yml
├── requirements.txt
├── LICENSE
└── README.md                 # This file
```

| File or directory | Purpose |
| --- | --- |
| `01_data_split_and_homology_control.ipynb` | Prepare datasets, construct seed-specific splits, and perform homology-control analyses. |
| `02_feature_encoding_training_and_benchmarking.ipynb` | Encode sequences, compare feature sets/classifiers, train MLP models, and generate evaluation outputs. |
| `03_blast_baseline_comparison.ipynb` | Run BLAST-based comparisons on the specified internal and independent datasets. |
| `04_unreviewed_sequence_prediction.ipynb` | Generate ProtT5 embeddings, make five-seed ensemble predictions, and export CKSAAP feature tables. |
| `data_input/` | Project input sequences and intermediate data files. |
| `split_seed_*/` | Seed-specific train/test data and generated features. |
| `saved_models_2026_eval/` | Files associated with the 2026 evaluation workflow; **do not assume they are interchangeable with final inference models**. |
| `CKSAAP_MRMD/` | Supplementary sequence-feature analysis materials. |
| `GO_VGIC_analysis_github/` | Supplementary GO annotation/analysis materials. |
| `environment.yml` | Exported Conda environment specification. |
| `requirements.txt` | Python dependency list. |

## Requirements

- A Linux environment is the closest match to the currently exported Conda configuration.
- Python 3.12 (the Conda file pins Python 3.12.8).
- PyTorch, Transformers, scikit-learn, NumPy, pandas, Biopython, and the other dependencies listed in `environment.yml` or `requirements.txt`.
- Internet access to download pretrained language models from Hugging Face, unless local model directories are supplied.
- Adequate memory/storage; a CUDA-enabled GPU is recommended for substantial ProtT5 embedding workloads. CPU execution is possible in the code but may be slow.
- **BLAST+** (`blastp`, `makeblastdb`) for the BLAST baseline notebook. **CD-HIT** is also needed for any clustering steps that invoke it.

> **Environment limitation:** The provided `environment.yml` is a platform-specific export with exact build constraints and mirror URLs. It has **not** been verified here as a portable, clean-install environment on every operating system. Check availability of the listed dependencies before relying on it for a new system.

## Installation

Clone the repository and run commands from its **root directory**:

```bash
git clone https://github.com/gaozhiw/VGICs-pred.git
cd VGICs-pred
```

### Option A: Conda (closest to the recorded environment)

```bash
conda env create -f environment.yml
conda activate voltage_class
```

The environment is named **`voltage_class`** in `environment.yml`. Some notebook headers still refer to `voltage_class2`, which is a local environment/kernel name from development; choose a kernel associated with the environment that you actually installed.

For example, if a Jupyter interface is not already available:

```bash
python -m pip install jupyterlab
python -m ipykernel install --user --name voltage_class --display-name "Python (VGICs-pred)"
jupyter lab
```

The JupyterLab installation above is optional and adds a frontend to the recorded environment.

### Option B: Install Python dependencies in an existing compatible environment

```bash
python -m pip install -r requirements.txt
```

**Important:** `requirements.txt` is a pinned export, not a minimal cross-platform dependency specification. PyTorch/CUDA and other platform-specific packages may require separate installation. Use this route only after checking compatibility with your operating system and hardware.

For the BLAST workflow, confirm these commands are available:

```bash
blastp -version
makeblastdb -version
```

If unavailable, install NCBI BLAST+ separately (for example through Bioconda in an appropriate Conda environment). Check any additional external executables required by the homology-control cells.

## Pretrained protein language models

The notebooks use the following default Hugging Face model identifiers:

| Representation | Default identifier | Configuration variable |
| --- | --- | --- |
| ProtT5 | [`Rostlab/prot_t5_xl_uniref50`](https://huggingface.co/Rostlab/prot_t5_xl_uniref50) | `PROTT5_PATH` |
| ProtBert | [`Rostlab/prot_bert`](https://huggingface.co/Rostlab/prot_bert) | `PROTBERT_PATH` |
| ESM2 | [`facebook/esm2_t6_8M_UR50D`](https://huggingface.co/facebook/esm2_t6_8M_UR50D) | `ESM2_PATH` |

Without environment variables, the code attempts to load these models using the listed identifiers. To use an already downloaded local model, set the corresponding variable **before opening Jupyter**:

```bash
export PROTT5_PATH="/path/to/local/prot_t5_xl_uniref50"
export PROTBERT_PATH="/path/to/local/prot_bert"
export ESM2_PATH="/path/to/local/esm2_t6_8M_UR50D"

jupyter lab
```

Replace the example paths with paths on your own computer. You do not need all three models to run ProtT5-only inference. See the corresponding model pages for upstream terms, citations, and hardware requirements.

## Workflows

The project currently exposes research workflows through Jupyter notebooks, **not a standalone `predict.py` command-line interface**.

### A. Data preparation and homology control

Open:

[`01_data_split_and_homology_control.ipynb`](01_data_split_and_homology_control.ipynb)

The notebook documents these example inputs:

```text
data_input/515_voltage.fasta
data_input/872_non_voltage.fasta
data_input/input_sequence_cdhit_0.4_2023.fasta
```

It prepares or analyzes seed-specific partitions and homology statistics. Its documented outputs include `split_seed_{seed}/` files, tabular sequence-identity summaries, and plot data.

Before re-running this notebook, check the input paths and any CD-HIT/BLAST executables used in individual cells. Save the exact seed-specific train/test membership used for the manuscript. Re-running partition or filtering operations can alter the evaluated samples if input versions or parameters change.

### B. Feature encoding, training, and benchmarking

Open:

[`02_feature_encoding_training_and_benchmarking.ipynb`](02_feature_encoding_training_and_benchmarking.ipynb)

This notebook contains several **separate experimental sections**, including language model embedding generation, feature/classifier comparisons, training, and evaluation. It is not a single small inference script; inspect the input/output configuration at the beginning of each section before executing it.

The final-selection MLP workflow writes paired artifacts to a directory named `saved_models/`:

```text
saved_models/
├── model_seed_42.pth
├── scaler_seed_42.pkl
├── model_seed_2023.pth
├── scaler_seed_2023.pkl
├── model_seed_1234.pth
├── scaler_seed_1234.pkl
├── model_seed_567.pth
├── scaler_seed_567.pkl
├── model_seed_100.pth
└── scaler_seed_100.pkl
```

This tree shows the **required naming convention for inference**, not a claim that these files are currently distributed in the repository.

**Critical:** Each `scaler_seed_*.pkl` must have been fitted on the same training data and for the same model version as its corresponding `model_seed_*.pth`. Do not substitute scalers from another evaluation stage. Some experimental sections load a file named `test_features.pkl` during model selection; verify its actual validation/test role against the accepted manuscript before interpreting independent test-set performance.

### C. BLAST baseline evaluation

Open:

[`03_blast_baseline_comparison.ipynb`](03_blast_baseline_comparison.ipynb)

This notebook uses the relevant splits, sequence data, and external BLAST+ binaries to run baseline comparisons. Documented outputs include:

```text
blast_baseline_internal_test.csv
blast_baseline_internal_test_summary.csv
blast_baseline_independent_test.csv
dl_ensemble_predictions_independent_test.csv
dl_wins_blast_fails.csv
```

These names describe notebook-generated outputs; the files may not be included in a fresh repository clone. Run only after the required inputs, paired model artifacts, and external tools have been prepared.

### D. Predict unreviewed sequences

Open:

[`04_unreviewed_sequence_prediction.ipynb`](04_unreviewed_sequence_prediction.ipynb)

**Required inputs:** The currently configured notebook expects:

```text
data_input/5551_add_515_872.fasta
data_input/prot5_5551_add_515_872.pkl   # generated by the first stage if needed
saved_models/model_seed_{seed}.pth
saved_models/scaler_seed_{seed}.pkl
```

The prediction workflow contains three stages:

1. **ProtT5 embedding generation.** Read FASTA sequences and generate 1,024-dimensional embeddings saved in the configured pickle file.
2. **Five-seed ensemble prediction.** Load the stored feature matrix, transform it with each seed's *training-fitted* scaler, load its corresponding MLP, and average the resulting probabilities.
3. **Optional CKSAAP analysis output.** Match the prediction table to configured `data_input/cat_0.4.fasta`, `cat_0.5.fasta`, `cat_0.6.fasta`, and `cat_0.7.fasta` datasets and export feature CSV files.

The main predicted-sequence output is:

```text
predictions_5551_ensemble.csv
```

Its columns are:

| Column | Meaning |
| --- | --- |
| `ID` | Protein identifier from the input FASTA/embedding records. |
| `Prob_Avg` | Mean prediction probability across five configured model seeds. |
| `Prediction` | Binary label; current notebook uses `Prob_Avg > 0.5` to assign class `1`. |
| `Prob_Seed_42`, etc. | Individual seed-specific prediction probabilities. |

A predicted label of `1` denotes a putative VGIC-class sequence; a predicted label of `0` denotes the alternative class under the trained model. The probabilities are model outputs, not experimental evidence or necessarily calibrated functional probabilities.

**Changing the prediction input:** The current notebook has configuration variables, rather than a general-purpose command-line argument parser. For another FASTA file, update `FASTA_INPUT`, `OUTPUT_PKL`, and `EMBEDDING_PKL` consistently and set a suitable `OUTPUT_CSV`. Ensure unique FASTA identifiers and valid amino-acid sequences. The code maps `U`, `Z`, `O`, and `B` to `X` for ProtT5 tokenization.

**Important model-file prerequisite:** The public repository contains a folder named `saved_models_2026_eval/`, while this prediction notebook looks for `saved_models/`. Do **not** rename or copy evaluation weights into the inference directory without verifying that they match the required MLP architecture, training stage, and scalers. Obtain the verified paired inference artifacts from the appropriate archival release, or regenerate them using the exact training workflow.

**Current implementation caution:** The notebook skips a model if a `.pth`/`.pkl` pair is absent, but its averaging array still reserves a column for that seed. Until corrected, **all five model/scaler pairs must be present**; otherwise the ensemble probability can be biased downward. Confirm that every seed was evaluated successfully before using or reporting predictions.

## Data, splits, and reproducibility

The study uses homology-controlled data preparation, multiple seeds, and specific test or discovery data. To reproduce a paper result, retain and report:

1. The **exact input sequences**, labels, accession identifiers, preprocessing rules, and source database release dates.
2. The **exact sequence-identity thresholds** and the cross-split homology-control procedure.
3. The five seed-specific train/test lists and all random seeds.
4. The **model identifiers/revisions**, embedding generation settings, model architecture, and trained weights.
5. Each matching fitted `StandardScaler` and the feature ordering.
6. Evaluation code, classifier thresholds, and input predictions used to calculate reported metrics.
7. Versions of Python, PyTorch, Transformers, scikit-learn, CD-HIT, BLAST+, and relevant packages.

The current notebooks and folders provide parts of this workflow. **Do not assume every accepted-manuscript result can be reproduced from the current public `main` branch without additional data or artifacts.** Files used for the DeepGOPlus and NetGO4.0 comparison in the revised manuscript should be cross-referenced here when the corresponding input/prediction files and versions have been verified.

The sequence-feature analyses are exploratory. Candidate sequences lacking selected existing functional annotations should be presented as **unvalidated predictions**, not experimentally established channels.

### Suggested minimal verification before creating a release

After preparing a clean environment and matching models, verify all of the following:

- Every documented input file is available at the expected path.
- Each of the five model weights has the correct matching scaler.
- ProtT5 feature vectors have the expected dimensionality (1,024 for the current pipeline).
- The output CSV has the expected columns, one row per input identifier, and probabilities between 0 and 1.
- The same sequence gives consistent results when predicted alone and together with other sequences, within numerical tolerance.
- Reproduced manuscript metrics and candidate counts are checked against the frozen accepted-manuscript results.
- No absolute paths to a developer's workstation are required.

Run inference only with **trusted** pickle/model files. Python pickle deserialization is unsafe for files from untrusted sources.

## Software and data availability

- **Live source repository:** https://github.com/gaozhiw/VGICs-pred
- **Versioned source releases:** https://github.com/gaozhiw/VGICs-pred/releases
- **Zenodo record for the accepted software and associated test data:** https://doi.org/10.5281/zenodo.24297507

The authors should add the verified archive DOI after creating the corresponding release and confirm that the archived content actually includes the software version and test data described in the publication. The code repository and the Zenodo archive serve different purposes: the GitHub branch may evolve; the versioned archive preserves a specific research artifact.

## Citation

If you use VGICs-pred, please cite the associated study:

> *VGICs-pred: a homology-controlled protein language model method for voltage-gated ion channel annotation and exploratory sequence-feature analysis*. **Bioinformatics** (accepted, 2026). **[ADD AUTHORS, JOURNAL ARTICLE DOI AND FINAL BIBLIOGRAPHIC DETAILS WHEN AVAILABLE]**
Also cite the upstream protein language models and other third-party methods as appropriate for the analysis performed. -->

## License

The source code in this repository is made available under the terms of the [MIT License](LICENSE). Third-party pretrained models, databases, datasets, and external tools may have separate licenses or attribution requirements; consult their respective sources.

## Support

For questions, suspected bugs, and reproducibility issues, use the repository's [GitHub Issues](https://github.com/gaozhiw/VGICs-pred/issues) page. When reporting an issue, include your operating system, Python/PyTorch versions, the notebook section, the relevant input format, and the full error message. Avoid sharing unpublished or sensitive sequence data in public issues.

---

### Maintainer checklist — complete before the next archived release

This section is intentionally retained until the manuscript-matched version is verified.

- [ ] Confirm a clean installation and align `environment.yml` with notebook kernel instructions.
- [ ] Verify that all five **inference** model/scaler pairs are available and correspond to the accepted study.
- [ ] Fix the missing-model behavior in `04_unreviewed_sequence_prediction.ipynb`.
- [ ] Confirm independent test-set handling and the use of validation data for model selection.
- [ ] Check the released data and scripts against the revised paper, including DeepGOPlus/NetGO4.0 comparisons.
- [ ] Provide a small verified FASTA input and expected output for a smoke test.
- [ ] Create a new versioned GitHub release and an associated Zenodo version containing the relevant source/test materials.
- [ ] Replace the Zenodo placeholder with the verified **version-specific DOI** and add the final paper citation when assigned.
