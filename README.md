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

This repository contains the trained model checkpoints and result artifacts for both datasets. It does not include training code — see the **Reproducibility** section below.

---

## Methodology Corrections (read before citing any numbers)

Two issues were found in the saved adversarial-robustness artifacts after initial publication and have been corrected in this repository. If you are citing results from an earlier version of this repo or the associated thesis draft, please use the corrected numbers below instead.

1. **Attack direction bug (both datasets).** The FGSM/PGD attack implementation perturbed each sample to *maximize* reconstruction error. This is backwards for a genuine evasion threat model — a real adversary wants to *minimize* reconstruction error so malicious traffic is misclassified as benign. Experiments re-evaluated with the corrected direction (`x_adv = x − ε·sign(∇loss)`): **Experiment 1 and Experiment 4 on BCCC-Mal-NetMem**, and **Experiment 3 and Experiment 4 on CICIoT2023**. Experiment 2 on both datasets, and Experiment 3 on BCCC-Mal-NetMem, have **not** been re-evaluated and still reflect the original, uncorrected attack direction — treat those numbers as unreliable. See the `_methodology_notice` field in each `adversarial_results.json` and the header of each `adversarial_comparison.txt`.
2. **Missing detection threshold (CICIoT2023, Experiment 4 only).** The saved checkpoint's threshold loaded as `None`, producing a degenerate always-flag-as-anomaly detector and the trivial "100% accuracy / 0% drop" figures seen in earlier versions of this repo. The threshold was back-solved from the checkpoint's own saved recall/FPR and cross-validated before re-evaluation.

The practical consequence: the adversarial-robustness effect of DP-SGD is **dataset-dependent, not a universal co-benefit**. On CICIoT2023, PPF-VAE is genuinely more robust than the non-federated Centralized+DP baseline. On BCCC-Mal-NetMem, PPF-VAE is genuinely *more vulnerable* than the non-private Centralized VAE baseline — roughly three times more, at the strongest tested FGSM setting. Diagnostic analysis traces this reversal to each model's detection threshold sitting at a different position relative to its own malicious-sample reconstruction-error distribution, not to any intrinsic property of DP-SGD training. See the full discussion in the thesis, §4.5.3 and §5.3.3.

---

## Datasets

Experiments are conducted on two benchmark network traffic datasets:

### CICIoT2023
Large-scale IoT network intrusion dataset with pre-split train/test/validation partitions.

| Split | Benign Samples | Attack Samples | Features | Attack Classes |
|-------|:--------------:|:--------------:|:--------:|:--------------:|
| Train | 129,538 | — | 46 | 33 |
| Validation | 27,519 | — | 46 | 33 |
| Test | 27,709 | 1,149,142 | 46 | 33 |

🔗 Source: [CIC IoT Dataset 2023](https://www.unb.ca/cic/datasets/iotdataset-2023.html)

### BCCC-Mal-NetMem
Multi-class malware network memory dataset with highly imbalanced class distribution.

- ~3.4 GB of network flow CSVs across multiple malware families
- Benign-to-malicious ratio makes this a challenging real-world scenario

🔗 Source: [BCCC Datasets — University of New Brunswick](https://www.unb.ca/cic/datasets/)

> **Note:** Raw CSV files are not stored in this repository. See `Data Load.txt` / `Data load.txt` in each dataset folder for the preprocessing steps.

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
│   ├── adversarial_results.json          # FGSM & PGD robustness (raw; see methodology notice)
│   ├── adversarial_comparison.txt        # Adversarial summary table (corrected)
│   ├── comprehensive_comparison.png      # Multi-metric bar chart
│   ├── comparison.png                    # Accuracy/F1 comparison
│   ├── adversarial_robustness.png        # Attack robustness plot (corrected)
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
    ├── adversarial_results.json          # (see methodology notice)
    ├── adversarial_comparison.txt        # (corrected)
    ├── comprehensive_comparison.png
    ├── comparison.png
    ├── adversarial_robustness.png        # (corrected)
    ├── non_iid_distribution.png
    ├── non_iid_metrics.json
    └── Data load.txt
```

---

## Results

### Detection Performance

**CICIoT2023**

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

### Adversarial Robustness (FGSM & PGD Attacks) — corrected, see note above

**CICIoT2023** — Centralized+DP (Exp 3) vs. PPF-VAE (Exp 4), strongest tested FGSM perturbation (ε=0.20)

| Model | Clean Acc | Adv Acc | Accuracy Drop | Attack Success Rate |
|-------|:---------:|:-------:|:--------------:|:--------------------:|
| Centralized + DP | 92.26% | 81.66% | 10.60 pp | 12.05% |
| **PPF-VAE** | 59.03% | 56.48% | **2.55 pp** | **3.90%** |

> PPF-VAE is genuinely more robust here — a **76% reduction in accuracy drop** and a **68% reduction in attack success rate** relative to Centralized+DP. (Experiments 1 and 2 on this dataset have not been evaluated under the corrected attack direction.)

**BCCC-Mal-NetMem** — Centralized VAE (Exp 1) vs. PPF-VAE (Exp 4), strongest tested FGSM perturbation (ε=0.20)

| Model | Clean Acc | Adv Acc | Accuracy Drop | Attack Success Rate |
|-------|:---------:|:-------:|:--------------:|:--------------------:|
| Centralized VAE | 50.96% | 46.56% | 4.40 pp | 4.40% |
| **PPF-VAE** | 48.88% | 35.50% | **13.38 pp** | **16.21%** |

> PPF-VAE is roughly **three times more vulnerable** here, not less — the opposite of what the uncorrected attack direction originally suggested. Diagnostic analysis traces this to PPF-VAE's detection threshold sitting in a more densely populated region of the malicious-sample reconstruction-error distribution, leaving more correctly-detected malicious samples only marginally above the decision boundary. (Experiments 2 and 3 on this dataset have not been evaluated under the corrected attack direction.)

**Takeaway:** the "DP-SGD hardens the decision boundary" effect is real on one dataset and reversed on the other. It depends on where each independently-calibrated detection threshold happens to fall relative to that dataset's malicious-error distribution — not on an intrinsic robustness property of differential privacy itself.

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
- FL clients: 3 | Non-IID distribution: enabled (α=0.5)
- FL rounds: 3 (non-private Federated VAE baseline) / 7 (optimized PPF-VAE)
- Feature scaling: z-score (StandardScaler), fit on the benign training partition, for both datasets
- DP: per-sample gradient clipping + Gaussian mechanism (Rényi DP accounting)
- Detection threshold: tuned on benign validation set (97th percentile reconstruction error)
- Adversarial evaluation: FGSM and PGD at ε ∈ {0.01, 0.05, 0.10, 0.20}

---

## Reproducibility

This repository is a **results archive**, not a runnable training pipeline — the notebook used to produce these checkpoints and results is not included here. To load and inspect a checkpoint:

```bash
pip install torch numpy
```

```python
import torch
ck = torch.load("CICIOT23/exp4_ppfvae_optimized.pth", map_location="cpu", weights_only=False)
print(ck["threshold"], ck["metrics"])
```

Each `.pth` file contains the model's `state_dict`, its calibrated anomaly-detection `threshold`, and a `metrics` dict with the reported accuracy/precision/recall/F1/AUC-ROC. See the thesis for the full model architecture and training configuration needed to reconstruct the `nn.Module` class before loading `state_dict` into it.

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
