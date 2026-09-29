# DeepDTA-Extended
DeepDTA reproduction + GNN/BiLSTM extension
# DeepDTA Extended

Reproduction and extension of **DeepDTA** (Öztürk et al., 2018) for drug-target binding affinity prediction.

## Goal
Replace DeepDTA's CNN encoders with:
- **Drug:** Graph Neural Network (GIN)
- **Protein:** BiLSTM + Attention

## Evaluation
- Standard random splits
- Cold-start splits (new drug, new target, both new)
- Metrics: CI, MSE, RMSE, Spearman, r²m, AUPR
- Statistical significance testing

## Team
- Havisha — Technical Lead (architecture, interpretability, cold-start)
- Charan — Integration & Training
- Abhi — Drug GNN
- Rish — Protein BiLSTM + Attention
- Hari — Evaluation & Metrics

## Setup
```bash
git clone git@github.com:havisha07/DeepDTA-Extended.git
cd DeepDTA-Extended
pip install -r requirements.txt
