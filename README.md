# DeepDTA Reproduction

Faithful reproduction of **DeepDTA** — a deep learning model that predicts drug-target binding affinity using only sequence data.

Based on: Öztürk, Özgür, Özkirimli (2018), *"DeepDTA: deep drug-target binding affinity prediction"*, Bioinformatics.

## Project Overview

Drug discovery requires testing millions of drug-protein pairs in a lab. This is slow and expensive. DeepDTA predicts binding affinity computationally using deep learning — taking a drug's SMILES string and a protein's amino acid sequence as input, and outputting a predicted binding strength (pKd).

Our project reproduces this model on two standard benchmark datasets and validates the results against the published paper.

## Results

### Reproduction on Davis Dataset
| Metric | Paper | Ours |
|---|---|---|
| Test CI | 0.878 | **0.8911** |
| Test MSE | 0.261 | **0.2147** |

### Reproduction on KIBA Dataset
| Metric | Paper | Ours |
|---|---|---|
| Test CI | 0.863 | **0.8762** |
| Test MSE | 0.194 | **0.1765** |

Both baselines match and slightly exceed the paper's reported values.

## Tech Stack

- **Language:** Python 3.10
- **Framework:** TensorFlow 2.x / Keras
- **Model:** Convolutional Neural Networks (CNNs) for both drug and protein branches
- **Inputs:** SMILES strings (drugs), amino acid sequences (proteins)
- **Data:** Davis Kinase, KIBA
- **Training:** Google Colab (Tesla T4 GPU)
- **API:** FastAPI (for serving predictions) — in progress
- **Frontend:** HTML/CSS/JavaScript (demo interface) — in progress

## Model Architecture

DeepDTA uses two parallel CNN branches that learn representations from raw sequence data:

**Drug Branch (SMILES):**
Embedding(128) → Conv1D(32, k=4) → Conv1D(64, k=6) → Conv1D(96, k=8) → GlobalMaxPool

**Protein Branch (amino acid sequence):**
Embedding(128) → Conv1D(32, k=4) → Conv1D(64, k=8) → Conv1D(96, k=12) → GlobalMaxPool

**Fusion:**
Concat(192) → Dense(1024) → Dropout(0.1) → Dense(1024) → Dropout(0.1) → Dense(512) → Dense(1)


Total trainable parameters: **1,968,897**

## Datasets

| Dataset | Drugs | Proteins | Interactions |
|---|---|---|---|
| Davis | 68 | 442 | 30,056 |
| KIBA | 2,111 | 229 | 118,254 |

Source data obtained from the original DeepDTA repository: https://github.com/hkmztrk/DeepDTA

## Training Setup

- Loss: MSE
- Optimizer: Adam, learning rate = 0.001
- Batch size: 256
- Epochs: 100
- Sequence lengths: 85 (Davis drug), 1200 (Davis protein), 100 (KIBA drug), 1000 (KIBA protein)

## Team

| Member | Role |
|---|---|
| Havisha | Technical Lead — model training, evaluation, report |
| Charan | Integration — training pipeline, API backend |
| Abhi | Data & Preprocessing — dataset loading, encoding |
| Rish | Model Verification — architecture validation |
| Hari | Evaluation — metrics, statistical analysis |

## Repository Structure
notebooks/ Working Colab notebook for training
src/ Clean Python modules (encoding, model, training, metrics)
results/ Trained model metrics (Davis, KIBA)
docs/ Architecture spec and findings
figures/ Plots for report

## How to Reproduce

1. Clone this repo
2. Install dependencies: `pip install -r requirements.txt`
3. Download Davis and KIBA data from the original DeepDTA repo
4. Run the training notebook in `notebooks/`

## Status

- Data pipeline for Davis and KIBA [DONE]
- DeepDTA model implemented in TensorFlow [DONE]
- Both baselines reproduced (beat the paper) [DONE]
- 5-fold cross-validation (paper's exact setup)
- All 4 evaluation metrics (CI, MSE, r²m, AUPR)
- FastAPI backend + HTML frontend
- Final report


# DeepDTA Architecture

Reproduction of the model described in Öztürk et al., 2018.

## Inputs

- **Drug:** SMILES string, encoded to integers using a 64-character dictionary, padded/truncated to length 85 (Davis) or 100 (KIBA).
- **Protein:** Amino acid sequence, encoded to integers using a 25-character dictionary, padded/truncated to length 1200 (Davis) or 1000 (KIBA).

## Drug Branch (CNN)

| Layer | Params |
|---|---|
| Embedding | vocab=65, dim=128 |
| Conv1D | 32 filters, kernel=4, ReLU |
| Conv1D | 64 filters, kernel=6, ReLU |
| Conv1D | 96 filters, kernel=8, ReLU |
| GlobalMaxPooling1D | — |

## Protein Branch (CNN)

| Layer | Params |
|---|---|
| Embedding | vocab=26, dim=128 |
| Conv1D | 32 filters, kernel=4, ReLU |
| Conv1D | 64 filters, kernel=8, ReLU |
| Conv1D | 96 filters, kernel=12, ReLU |
| GlobalMaxPooling1D | — |

## Fusion Head

| Layer | Params |
|---|---|
| Concatenate | 96 + 96 = 192 |
| Dense | 1024, ReLU, Dropout 0.1 |
| Dense | 1024, ReLU, Dropout 0.1 |
| Dense | 512, ReLU |
| Dense | 1 (output) |

## Training

- Loss: Mean Squared Error
- Optimizer: Adam, lr=0.001
- Batch size: 256
- Epochs: 100
- Metric monitored: Concordance Index (CI) on validation set

## Key Implementation Details

- **pKd conversion (Davis):** `pKd = -log10(Kd / 1e9)` where Kd is in nanomolar
- **KIBA scores** used directly (no transformation)
- **CI callback** saves the best model based on validation CI
- **Best val CI:** Davis = 0.9002 (epoch 97), KIBA = 0.8762 (epoch 100)
File 6: docs/findings.md
markdown
# Findings

## Summary

We successfully reproduced DeepDTA on both benchmark datasets. Our results match and slightly exceed the published paper.

## Results

### Davis
- **Test CI:** 0.8911 (paper: 0.878)
- **Test MSE:** 0.2147 (paper: 0.261)

### KIBA
- **Test CI:** 0.8762 (paper: 0.863)
- **Test MSE:** 0.1765 (paper: 0.194)

## Observations

1. **Training converged smoothly** — no overfitting observed in the first 100 epochs.
2. **Best val CI on Davis** was 0.9002 at epoch 97, indicating the model had mostly converged by epoch 60.
3. **KIBA is 4× larger** than Davis but follows the same convergence pattern with slightly lower final CI.
4. **Our reproduction slightly exceeds the paper** — likely due to random seed variation and modern TensorFlow optimizer behavior. The difference is within expected variance.

## TO DO:

1. Run 5-fold cross-validation to match the paper's methodology exactly.
2. Implement the remaining metrics (r²m, AUPR).
3. Build the FastAPI backend and HTML frontend for the demo.
4. Write the final report.
