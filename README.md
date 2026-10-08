# PPF-VAE: Privacy-Preserving Federated Variational Autoencoder for Network Intrusion Detection

> **Master's Thesis** — Evaluating the trade-off between privacy, utility, and adversarial robustness in federated anomaly detection across six real-world network traffic datasets.

---

## Overview

This repository contains the experimental results, trained models, and evaluation artifacts for a thesis investigating **PPF-VAE** — a Privacy-Preserving Federated Variational Autoencoder designed for network intrusion detection.

The framework combines:
- **Federated Learning (FL)** — distributed training across multiple clients without sharing raw data
- **Differential Privacy (DP)** — formal privacy guarantees via Gaussian noise injection
- **Variational Autoencoder (VAE)** — unsupervised anomaly detection trained on benign traffic only

Four experimental configurations are systematically compared across six datasets:

| # | Experiment | Privacy | Distribution |
|---|-----------|---------|-------------|
| 1 | Centralized VAE | ❌ None | Centralized |
| 2 | Federated VAE (FL) | ❌ None | Federated |
| 3 | Centralized + DP (ε=10) | ✅ DP | Centralized |
| 4 | **PPF-VAE (Optimized)** | ✅ DP | Federated |

---

## Datasets

Six benchmark network traffic datasets are evaluated, covering a wide range of IoT, 5G, and enterprise network environments:

| Dataset | Domain | Size |
|---------|--------|------|
| [CICIOT23](https://www.unb.ca/cic/datasets/iotdataset-2023.html) | IoT Network Traffic | ~5.5M train rows, 46 features, 34 attack classes |
| [5G-NIDD](https://www.kaggle.com/datasets/ramoliyafenil/5g-network-intrusion-detection-dataset) | 5G Network Intrusion | Combined + synthetic 100k samples |
| [CICIDS2017](https://www.unb.ca/cic/datasets/ids-2017.html) | Enterprise Network IDS | Synthetic 50k realistic samples |
| [CICIDS2018](https://www.unb.ca/cic/datasets/ids-2018.html) | Enterprise Network IDS | Synthetic 100k samples |
| [CICIOMT2024](https://www.unb.ca/cic/datasets/iomt-dataset-2024.html) | IoMT Healthcare Network | Synthetic 100k samples |
| [BCCC-Mal-NetMem](https://www.unb.ca/cic/datasets/) | Malware Network Memory | 3.4 GB multi-class CSV data |

> **Note:** Raw CSV dataset files are not stored in this repository due to size constraints. See the links above or the `Data Load.txt` file in each dataset folder for loading instructions.

---

## Repository Structure

```
PPF-VAE-Thesis/
├── 5G-NIDD/
│   ├── exp1_centralized_vae.pth          # Trained model weights
│   ├── exp2_federated_vae.pth
│   ├── exp3_centralized_dp.pth
│   ├── exp4_ppfvae_optimized.pth
│   ├── comparison.json                   # Accuracy/F1/AUC results
│   ├── adversarial_results.json          # FGSM & PGD robustness
│   ├── adversarial_comparison.txt
│   ├── comprehensive_comparison.png      # Visualisation
│   ├── non_iid_distribution.png
│   └── non_iid_metrics.json
├── BCCC-Mal-NetMem/                      # Same structure as above
├── CICIDS2017/
├── CICIDS2018/
├── CICIOMT24/
└── CICIOT23/
    ├── exp1_metrics.json                 # Per-experiment detail
    ├── exp2_metrics.json
    ├── exp3_metrics.json
    ├── exp4_metrics.json
    ├── experiments_comparison.json
    ├── zero_day_analysis.png             # Zero-day detection
    ├── zero_day_metrics.csv
    └── all_confusion_matrices.png
```

---

## Key Results

### Detection Performance (AUC-ROC)

| Dataset | Centralized VAE | Federated VAE | Centralized+DP | **PPF-VAE** |
|---------|:--------------:|:-------------:|:--------------:|:-----------:|
| CICIOT23 | 0.949 | 0.948 | 0.968 | **0.978** |
| 5G-NIDD | 0.9998 | 1.000 | 1.000 | **1.000** |
| CICIDS2017 | 0.9998 | 0.9997 | 0.9994 | **0.9997** |
| CICIDS2018 | — | — | — | — |
| CICIOMT24 | — | — | — | — |
| BCCC-Mal-NetMem | 0.881 | 0.790 | 0.826 | **0.841** |

### Privacy-Utility Trade-off (BCCC-Mal-NetMem)

| Model | Privacy Budget (ε) | F1-Score | Utility Loss vs Baseline |
|-------|:-----------------:|:--------:|:------------------------:|
| Centralized VAE | No DP | 0.5296 | — |
| Federated VAE | No DP | 0.4317 | 18.5% |
| Centralized + DP | ε = 10.39 | 0.4324 | 18.4% |
| **PPF-VAE** | **ε = 5.96** | **0.4973** | **6.1%** |

> PPF-VAE achieves stronger privacy (lower ε) with significantly less utility loss than Centralized+DP.

### Adversarial Robustness (CICIOT23 — FGSM & PGD Attacks)

| Model | FGSM ε=0.2 Acc Drop | PGD ε=0.1 Acc Drop |
|-------|:-------------------:|:------------------:|
| Centralized + DP | −5.20% | −0.37% |
| **PPF-VAE** | **0.00%** | **0.00%** |

PPF-VAE achieves **perfect adversarial robustness** on CICIOT23, with 0% accuracy drop under all tested FGSM and PGD attack strengths.

---

## Methodology

```
┌─────────────────────────────────────────────────────────────┐
│                        PPF-VAE Framework                    │
│                                                             │
│  Client 1 ──┐                                               │
│  Client 2 ──┼──► Federated Aggregation ──► Global VAE      │
│  Client 3 ──┘         (FedAvg)               Model         │
│       │                                        │            │
│  Local DP Noise                        Anomaly Scoring      │
│  (Gaussian Mech.)                   (Reconstruction Error)  │
└─────────────────────────────────────────────────────────────┘
```

- **Model:** VAE with encoder/decoder MLP, latent dimension tuned per dataset
- **Federated Setup:** 3 clients, 3 rounds, Non-IID data distribution
- **Privacy:** Per-sample gradient clipping + Gaussian noise (Rényi DP accounting)
- **Detection:** Threshold on reconstruction error (trained on benign traffic only)
- **Adversarial Evaluation:** FGSM and PGD attacks at multiple ε strengths

---

## Requirements

```bash
pip install torch torchvision numpy pandas scikit-learn matplotlib opacus
```

- Python 3.8+
- PyTorch (CUDA recommended for large datasets)
- [Opacus](https://opacus.ai/) for differential privacy

---

## Citation

If you use this work, please cite:

```bibtex
@mastersthesis{akoh2026ppfvae,
  title     = {PPF-VAE: Privacy-Preserving Federated Variational Autoencoder for Network Intrusion Detection},
  author    = {Akoh, Aire-Blessing},
  year      = {2026},
  school    = {[Your Institution]},
}
```

---

## License

This repository is for academic research purposes. Dataset usage is subject to the respective dataset providers' terms of use.
