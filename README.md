# DCASE2025_TASK2

This repository contains the code, extracted feature files, and evaluation scripts for a Topological Data Analysis (TDA)-based unsupervised anomalous sound detection framework developed for DCASE 2025 Task 2: First-Shot Unsupervised Anomalous Sound Detection for Machine Condition Monitoring.

## Overview

The proposed method detects abnormal machine sounds without using anomaly samples during model development. Each WAV recording is converted into a Mel spectrogram, interpreted as a two-dimensional image, and transformed into a cubical complex. Persistent homology is then used to extract topological descriptors from the Mel-spectrogram structure.

The extracted features include lifetime statistics, birth/death statistics, Betti curve features, persistence landscapes, persistence silhouettes, persistence images, Carlsson coordinates, and H0/H1 interaction measures. Machine-specific feature subsets are selected and used with a k-nearest-neighbor distance-based anomaly scoring strategy.

## Repository Contents

* `ORJ_DCASE_TDA_FEATURE_CREATOR.ipynb`
  Creates cubical-complex-based TDA features from Mel spectrograms.

* `ORJ_DCASE_FEATURE_SELECTION.ipynb`
  Applies machine-specific feature selection using a genetic algorithm.

* `ORJ_DCASE_EVALUATION_FILES.ipynb`
  Generates DCASE-style anomaly score and decision result CSV files.

* `ORJ_DCASE_EVALUATOR_GROUNDTRUTH.ipynb`
  Runs the official DCASE 2025 Task 2 evaluator using the generated outputs.

* `Best_Features_by_Machine_Type.txt`
  Contains the selected TDA feature subsets for each machine type.

* `cubical_mel_tda_features_*.xlsx`
  Extracted TDA feature matrices for the evaluated machine types.

* `cubical_mel_tda_features_dev_*.xlsx`
  Development-set feature matrices for additional machine categories.

## Method Summary

1. Load machine-sound WAV recordings.
2. Convert each recording into a Mel spectrogram.
3. Transform the Mel spectrogram into a cubical complex.
4. Compute persistent homology for H0 and H1.
5. Extract topological feature families.
6. Select compact machine-specific feature subsets.
7. Compute kNN-distance-based anomaly scores.
8. Create DCASE-compatible output files.
9. Evaluate results with the official DCASE Task 2 evaluator.

## Dataset

This project is designed for the DCASE 2025 Task 2 first-shot unsupervised anomalous sound detection dataset. The raw dataset is not included in this repository and should be obtained from the official DCASE Challenge source.

## Usage

Clone the repository:

```bash
git clone https://github.com/korkutanapa/DCASE2025_TASK2.git
cd DCASE2025_TASK2
```

Open the notebooks in Jupyter Notebook or Google Colab and run them in the following order:

1. `ORJ_DCASE_TDA_FEATURE_CREATOR.ipynb`
2. `ORJ_DCASE_FEATURE_SELECTION.ipynb`
3. `ORJ_DCASE_EVALUATION_FILES.ipynb`
4. `ORJ_DCASE_EVALUATOR_GROUNDTRUTH.ipynb`

## Requirements

The notebooks require standard Python data science and audio-processing libraries, including:

```bash
pip install numpy pandas scipy scikit-learn librosa matplotlib openpyxl
```

Additional TDA-related packages may be required depending on the notebook environment.

## Output Files

The evaluation pipeline creates DCASE-compatible files such as:

```text
anomaly_score_{MachineType}_section_00_test.csv
decision_result_{MachineType}_section_00_test.csv
```

These files can be evaluated using the official DCASE 2025 Task 2 evaluator.

## Citation

If you use this repository, please cite the related manuscript:

**Enhancing First-Shot Anomalous Sound Detection in Noisy Industrial Environments**

## Author

Korkut Anapa
Institute of Applied Mathematics
Middle East Technical University
Ankara, Türkiye
