# DCASE2025_TASK2

Topological Data Analysis (TDA) based unsupervised anomalous sound detection for **DCASE 2025 Challenge Task 2: First-Shot Unsupervised Anomalous Sound Detection for Machine Condition Monitoring**.

This repository contains notebooks, extracted feature files, AutoEncoder experiments, and DCASE-style output generation scripts for normal-only anomaly detection using topological descriptors extracted from machine-sound recordings.

Repository: <https://github.com/korkutanapa/DCASE2025_TASK2>

---

## 1. Project Overview

The goal of this project is to detect anomalous machine sounds under the first-shot unsupervised anomaly detection setting. The core idea is to transform each audio clip into a topological feature vector and then learn normal machine behavior using normal-only models.

The workflow is:

1. Load machine-sound WAV files.
2. Convert each audio clip into a Mel-spectrogram representation.
3. Extract TDA descriptors from cubical persistence, mainly using \(H_0\)- and \(H_1\)-based features.
4. Save the resulting feature matrices as Excel files.
5. Train normal-only anomaly detection models using only normal training samples.
6. Generate DCASE-compatible anomaly score and decision files.
7. Evaluate outputs using the official DCASE Task 2 evaluator.

The repository focuses especially on a normal-only AutoEncoder (AE) framework and development experiments used to select the final AE scoring strategy.

---

## 2. Challenge Setting

DCASE 2025 Task 2 is an unsupervised anomalous sound detection task for machine condition monitoring. The task requires training models using only normal sounds and then detecting unknown anomalous sounds in unlabeled test data.

The challenge includes three dataset types:

- **Development dataset**: contains known machine types with normal training clips and labeled normal/anomalous test clips. In this project, it is used for model design and validation.
- **Additional training dataset**: contains unseen evaluation machine types. Each machine section includes normal source-domain training clips, normal target-domain training clips, and supplementary clips.
- **Evaluation dataset**: contains 200 unlabeled test clips per machine section. These files do not provide normal/anomaly labels or source/target domain labels.

The final model is therefore trained and calibrated only from normal training data and then applied to unlabeled evaluation data.

---

## 3. Repository Contents

The repository contains the following main files.

### Main notebooks

| File | Description |
|---|---|
| `ORJ_DCASE_TDA_FEATURE_CREATOR.ipynb` | Notebook for extracting TDA-based features from audio data. |
| `ORJ_DCASE_AE.ipynb` | Main AutoEncoder-based normal-only anomaly scoring notebook. |
| `AE_RESULTS_DEV_BEARING.ipynb` | Development AE experiments for the bearing machine type. |
| `AE_RESULTS_DEV_FAN.ipynb` | Development AE experiments for the fan machine type. |
| `AE_RESULTS_DEV_GEARBOX.ipynb` | Development AE experiments for the gearbox machine type. |
| `AE_RESULTS_DEV_SLIDER.ipynb` | Development AE experiments for the slide rail machine type. |
| `AE_RESULTS_DEV_TOYCAR.ipynb` | Development AE experiments for ToyCar. |
| `AE_RESULTS_DEV_TOYTRAIN.ipynb` | Development AE experiments for ToyTrain. |
| `AE_RESULTS_DEV_VALVE.ipynb` | Development AE experiments for valve. |

### Additional/evaluation feature files

Files with the `_thr` suffix are used as normal training feature files. Files without `_thr` are used as test/evaluation feature files.

Examples:

```text
cubical_mel_tda_features_AutoTrash_thr.xlsx       # normal training features
cubical_mel_tda_features_AutoTrash.xlsx           # unlabeled test/evaluation features
cubical_mel_tda_features_BandSealer_thr.xlsx
cubical_mel_tda_features_BandSealer.xlsx
...
```

### Development feature files

Development files are stored as train/test pairs, for example:

```text
cubical_mel_tda_features_dev_bearing_train.xlsx
cubical_mel_tda_features_dev_bearing_test.xlsx
cubical_mel_tda_features_dev_fan_train.xlsx
cubical_mel_tda_features_dev_fan_test.xlsx
...
```

The development test files include labels and are used only for model selection and ROC-AUC analysis.

---

## 4. Methodology

### 4.1 TDA Feature Extraction

Each audio clip is first converted into a Mel-spectrogram. Cubical persistence is then applied to obtain topological information from the spectrogram surface. The extracted descriptors include feature families such as:

- persistence entropy,
- birth/death statistics,
- lifetime statistics,
- persistence images,
- persistence landscapes,
- silhouettes,
- Betti-curve descriptors,
- dominance/tail-share features,
- \(H_0\), \(H_1\), and \(H_1/H_0\) interaction descriptors.

The final output of this step is a tabular feature matrix where each row corresponds to one audio clip.

### 4.2 Normal-Only AutoEncoder Anomaly Detection

The AutoEncoder is trained only on normal training samples. The normal training feature matrix is split into:

- normal training subset,
- normal validation/calibration subset.

Preprocessing is fitted only on the normal training subset:

- median imputation,
- standard scaling.

The validation subset is used for score calibration and threshold estimation. Evaluation/test samples are never used during training or calibration.

### 4.3 Selected AE Structure

Development experiments showed that different machine types benefit from different AE scoring mechanisms. Therefore, the final approach uses a dev-guided multi-branch AE ensemble rather than a single AE score.

The final ensemble contains three AE branches:

1. **Compact sparse latent AE**
   - latent dimension: 4
   - noise: 0.0
   - sparse activity regularization: \(10^{-5}\)
   - main score: latent-space kNN percentile score

2. **Sparse denoising AE**
   - latent dimension: 16
   - Gaussian noise: 0.03
   - sparse activity regularization: \(10^{-5}\)
   - main scores: top-k reconstruction deviation and z-mean deviation

3. **Reconstruction-oriented AE**
   - latent dimension: 8
   - Gaussian noise: 0.03
   - sparse activity regularization: 0.0
   - main scores: MSE and MAE reconstruction percentiles

Each branch follows the encoder-decoder structure:

```text
input_dim -> 128 -> 64 -> latent_dim -> 64 -> 128 -> input_dim
```

with ReLU activation, batch normalization, dropout, and L2 kernel regularization.

### 4.4 Final Score Fusion

Raw anomaly scores are converted into percentile scores using the normal validation score distribution. The final anomaly score is computed as:

```text
final_score =
    0.45 * latent_knn_pct
  + 0.30 * topk_zmean_pct
  + 0.10 * zmean_pct
  + 0.10 * mse_pct
  + 0.05 * mae_pct
```

The decision threshold is estimated from the 95th percentile of the final validation-normal score distribution. The continuous anomaly score is used for AUC-based evaluation, while the thresholded score is used to create the decision file.

---

## 5. Output Format

For each machine type, the AE notebook creates two official DCASE-style CSV files:

```text
anomaly_score_{MachineType}_section_00_test.csv
decision_result_{MachineType}_section_00_test.csv
```

Both files are saved **without headers**.

Example anomaly score file:

```text
section_00_0000.wav,0.7321
section_00_0001.wav,0.1845
```

Example decision file:

```text
section_00_0000.wav,1
section_00_0001.wav,0
```

---

## 6. Installation

The notebooks were developed in Google Colab. A GPU runtime, such as T4, is recommended but not strictly required for the tabular AE experiments.

Install the required Python packages:

```python
!pip install -q numpy pandas scipy scikit-learn matplotlib openpyxl tensorflow keras joblib
```

Main dependencies:

- Python 3.x
- NumPy
- Pandas
- SciPy
- scikit-learn
- TensorFlow / Keras
- OpenPyXL
- Matplotlib
- Joblib

---

## 7. Usage

### Step 1: Extract TDA features

Run:

```text
ORJ_DCASE_TDA_FEATURE_CREATOR.ipynb
```

This notebook produces Excel feature files for each machine type.

### Step 2: Run development AE experiments

Use the development notebooks to compare AE structures and score types:

```text
AE_RESULTS_DEV_BEARING.ipynb
AE_RESULTS_DEV_FAN.ipynb
AE_RESULTS_DEV_GEARBOX.ipynb
AE_RESULTS_DEV_SLIDER.ipynb
AE_RESULTS_DEV_TOYCAR.ipynb
AE_RESULTS_DEV_TOYTRAIN.ipynb
AE_RESULTS_DEV_VALVE.ipynb
```

These notebooks are used for development analysis and model selection.

### Step 3: Generate final DCASE output files

Run:

```text
ORJ_DCASE_AE.ipynb
```

This notebook reads the `_thr.xlsx` normal training feature files and the corresponding test feature files, trains the normal-only AE ensemble, and creates DCASE-compatible output files.

### Step 4: Run official evaluator

After generating the score and decision files, use the official DCASE Task 2 evaluator to calculate the final metrics.

---

## 8. Machine Types Included

Additional/evaluation machine types included in the repository:

- AutoTrash
- BandSealer
- CoffeeGrinder
- HomeCamera
- Polisher
- ScrewFeeder
- ToyPet
- ToyRCCar

Development machine types included in the repository:

- Bearing
- Fan
- Gearbox
- Slide rail / Slider
- ToyCar
- ToyTrain
- Valve

---

## 9. Notes

- The method is normal-only and unsupervised during training.
- Test labels are not used for training, calibration, or threshold selection.
- Development labels are used only for model analysis and selection.
- Evaluation test files are treated as unlabeled data.
- Supplementary data are not treated as anomalous samples.
- Final scores are generated using only normal training/calibration statistics.

---

## 10. Citation

If this repository is used in academic work, please cite the related DCASE 2025 Task 2 challenge page and the corresponding paper/report associated with this project.

```bibtex
@misc{DCASE2025_TASK2_REPOSITORY,
  title  = {DCASE2025_TASK2: TDA-Based Unsupervised Anomaly Detection},
  author = {Anapa, Korkut},
  year   = {2026},
  url    = {https://github.com/korkutanapa/DCASE2025_TASK2}
}
```

---

## 11. License

No license file was identified in the repository at the time of writing. Please add an explicit license file if the repository will be shared or reused publicly.
