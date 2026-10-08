# PPF-VAE: Privacy-Preserving Federated VAE for Network Intrusion Detection

> **Master's Thesis** — Evaluating the trade-off between privacy, utility, and adversarial robustness in federated anomaly-based intrusion detection.

---

## Overview

**PPF-VAE** is a Privacy-Preserving Federated Variational Autoencoder for network intrusion detection. It combines three complementary techniques into a single framework:

| Component | Role |
|-----------|------|
| **Variational Autoencoder (VAE)** | Unsupervised anomaly detection — trained on benign traffic only, flags anomalies via reconstruction error |
| **Federated Learning (FL)** | Distributed training across multiple clients without sharing raw network data |
| **Differential Privacy (DP)** | Formal privacy guarantees via per-sample gradient clipping and Gaussian noise injection |

Four experimental configurations are compared to isolate the contribution of each component:

| # | Experiment | Federated | DP |
|---|-----------|:---------:|:--:|
| 1 | Centralized VAE (baseline) | ❌ | ❌ |
| 2 | Federated VAE | ✅ | ❌ |
| 3 | Centralized VAE + DP (ε=10) | ❌ | ✅ |
| 4 | **PPF-VAE** (FL + DP, optimized) | ✅ | ✅ |

---

## Datasets

Experiments are conducted on two benchmark network traffic datasets:

### CICIOT23
Large-scale IoT network intrusion dataset with pre-split train/test/validation partitions.

| Split | Benign Samples | Attack Samples | Features | Attack Classes |
|-------|:--------------:|:--------------:|:--------:|:--------------:|
| Train | 129,538 | — | 46 | 34 |
| Validation | 27,519 | — | 46 | 34 |
| Test | 27,709 | 1,149,142 | 46 | 34 |

🔗 Source: [CIC IoT Dataset 2023](https://www.unb.ca/cic/datasets/iotdataset-2023.html)

### BCCC-Mal-NetMem
Multi-class malware network memory dataset with highly imbalanced class distribution.

- ~3.4 GB of network flow CSVs across multiple malware families
- Benign-to-malicious ratio makes this a challenging real-world scenario

🔗 Source: [BCCC Datasets — University of New Brunswick](https://www.unb.ca/cic/datasets/)

> **Note:** Raw CSV files are not stored in this repository. See `Data Load.txt` in each dataset folder for preprocessing instructions.

---

## Repository Structure

```
PPF-VAE-Thesis/
├── CICIOT23/
│   ├── exp1_centralized_vae.pth          # Trained model weights
│   ├── exp2_federated_vae.pth
│   ├── exp3_centralized_dp.pth
│   ├── exp4_ppfvae_optimized.pth
│   ├── comparison.json                   # Accuracy / F1 / AUC-ROC results
│   ├── experiments_comparison.json       # Per-experiment detailed metrics
│   ├── exp{1-4}_metrics.json             # Individual experiment breakdowns
│   ├── adversarial_results.json          # FGSM & PGD robustness (raw)
│   ├── adversarial_comparison.txt        # Adversarial summary table
│   ├── comprehensive_comparison.png      # Multi-metric bar chart
│   ├── comparison.png                    # Accuracy/F1 comparison
│   ├── adversarial_robustness.png        # Attack robustness plot
│   ├── all_confusion_matrices.png        # Confusion matrices (all 4 exps)
│   ├── non_iid_distribution.png          # Client data distribution
│   ├── non_iid_metrics.json
│   ├── zero_day_analysis.png             # Zero-day detection evaluation
│   ├── zero_day_metrics.csv
│   └── Load Data.txt                     # Dataset loading instructions
│
└── BCCC-Mal-NetMem/
    ├── exp1_centralized_vae.pth
    ├── exp2_federated_vae.pth
    ├── exp3_centralized_dp.pth
    ├── exp4_ppfvae_optimized.pth
    ├── comparison.json
    ├── adversarial_results.json
    ├── adversarial_comparison.txt
    ├── comprehensive_comparison.png
    ├── comparison.png
    ├── adversarial_robustness.png
    ├── non_iid_distribution.png
    ├── non_iid_metrics.json
    └── Data load.txt
```

---

## Results

### Detection Performance

**CICIOT23**

| Metric | Centralized VAE | Federated VAE | Centralized+DP | **PPF-VAE** |
|--------|:--------------:|:-------------:|:--------------:|:-----------:|
| Accuracy | 62.07% | 64.40% | 92.30% | 59.95% |
| Precision | 99.89% | 99.85% | 99.80% | **99.95%** |
| Recall | 61.22% | 63.63% | 92.29% | 59.01% |
| F1-Score | 75.91% | 77.73% | 95.90% | 74.21% |
| **AUC-ROC** | 94.93% | 94.79% | 96.76% | **97.78%** |
| Privacy ε | — | — | 10.83 | **5.33** |

> PPF-VAE achieves the **highest AUC-ROC (97.78%)** and the **strongest privacy (ε=5.33)** — outperforming Centralized+DP on both dimensions simultaneously.

**BCCC-Mal-NetMem**

| Metric | Centralized VAE | Federated VAE | Centralized+DP | **PPF-VAE** |
|--------|:--------------:|:-------------:|:--------------:|:-----------:|
| Accuracy | 95.87% | 94.63% | 94.62% | 95.50% |
| Precision | 55.14% | 41.77% | 41.73% | 50.73% |
| Recall | 50.95% | 44.67% | 44.86% | 48.76% |
| F1-Score | 52.96% | 43.17% | 43.24% | **49.73%** |
| AUC-ROC | 88.06% | 79.00% | 82.58% | **84.08%** |
| Privacy ε | — | — | 10.39 | **5.96** |

### Privacy–Utility Trade-off (BCCC-Mal-NetMem)

| Model | Privacy Budget (ε) | F1-Score | Utility Loss vs Baseline |
|-------|:-----------------:|:--------:|:------------------------:|
| Centralized VAE | No DP | 52.96% | — |
| Federated VAE | No DP | 43.17% | −18.5% |
| Centralized + DP | ε = 10.39 | 43.24% | −18.4% |
| **PPF-VAE** | **ε = 5.96** | **49.73%** | **−6.1%** |

> PPF-VAE provides **stronger privacy at roughly half the epsilon budget** of Centralized+DP, while recovering 12 percentage points of utility.

### Adversarial Robustness (FGSM & PGD Attacks)

**CICIOT23**

| Model | FGSM ε=0.01 | FGSM ε=0.05 | FGSM ε=0.20 | PGD ε=0.10 |
|-------|:-----------:|:-----------:|:-----------:|:----------:|
| Centralized+DP | −0.39% drop | −1.51% drop | −5.20% drop | −0.37% drop |
| **PPF-VAE** | **0.00%** | **0.00%** | **0.00%** | **0.00%** |

**BCCC-Mal-NetMem**

| Model | FGSM ε=0.01 | FGSM ε=0.20 | PGD ε=0.10 |
|-------|:-----------:|:-----------:|:----------:|
| Centralized VAE | −0.36% | −19.60% | −2.20% |
| Federated VAE | −1.13% | −8.61% | −1.22% |
| Centralized+DP | −0.77% | −15.71% | −5.49% |
| **PPF-VAE** | **−0.08%** | **−3.11%** | **−1.29%** |

> On CICIOT23, PPF-VAE achieves **perfect adversarial robustness** (0% accuracy drop across all tested FGSM and PGD attack strengths). On BCCC-Mal-NetMem, it is the most robust model of all four configurations.

---

## Methodology

```
                        PPF-VAE Training Pipeline
  ┌────────────────────────────────────────────────────────────────┐
  │                                                                │
  │  ┌──────────┐    Local VAE     ┌─────────────────────────┐    │
  │  │ Client 1 │ ──── train ────► │                         │    │
  │  ├──────────┤                  │   Federated Aggregation  │    │
  │  │ Client 2 │ ──── train ────► │       (FedAvg)          │───►│ Global VAE
  │  ├──────────┤                  │                         │    │   Model
  │  │ Client 3 │ ──── train ────► │                         │    │
  │  └──────────┘                  └─────────────────────────┘    │
  │       │                                                        │
  │  Per-sample gradient clipping + Gaussian DP noise             │
  │  (Rényi DP accounting → formal ε guarantee)                   │
  │                                                                │
  └────────────────────────────────────────────────────────────────┘

  Inference:  Input → Encoder → Latent z → Decoder → Reconstruction Error
              If error > threshold  →  ANOMALY (attack traffic detected)
```

**Key configuration:**
- FL clients: 3 | FL rounds: 3 | Non-IID distribution: enabled (α=0.5)
- DP: per-sample gradient clipping + Gaussian mechanism (Rényi DP accounting)
- Detection threshold: tuned on benign validation set (95th percentile reconstruction error)
- Adversarial evaluation: FGSM and PGD at ε ∈ {0.01, 0.05, 0.10, 0.20}

---

## Requirements

```bash
pip install torch numpy pandas scikit-learn matplotlib opacus
```

- Python 3.8+
- PyTorch with CUDA (recommended — CICIOT23 has ~7M rows)
- [Opacus](https://opacus.ai/) for differentially private training

---

## Citation

```bibtex
@mastersthesis{akoh2026ppfvae,
  title  = {PPF-VAE: Privacy-Preserving Federated Variational Autoencoder
            for Network Intrusion Detection},
  author = {Akoh, Aire-Blessing},
  year   = {2026},
  school = {[Your Institution]},
}
```

---

## License

This repository is for academic research. Dataset usage is subject to the terms of the respective dataset providers.
