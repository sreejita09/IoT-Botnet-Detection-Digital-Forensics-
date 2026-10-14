# 🔍 IoT Botnet Detection + Digital Forensics

![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Google Colab](https://img.shields.io/badge/Google%20Colab-Ready-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/scikit--learn-ML-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)
![TShark](https://img.shields.io/badge/TShark-PCAP%20Analysis-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

**A complete digital forensics and machine-learning pipeline for IoT botnet detection — no physical hardware required.**

*Runs entirely on Google Colab using the [CICIoT2023](https://huggingface.co/datasets/somnath0100/CICIoT2023Small) dataset (Mirai UDP-Plain variant).*

---

## 📖 Overview

Connected IoT devices are increasingly targeted by botnets such as **Mirai**, which exploit weak credentials and unpatched firmware to conscript devices into large-scale DDoS attacks. This notebook provides a structured, reproducible workflow covering:

- 🔬 **Digital forensics** — evidence hashing, chain-of-custody preservation, and PCAP-level traffic examination
- 🚨 **Intrusion detection** — optional Suricata IDS rule-matching pass
- 🤖 **Supervised ML** — multi-model comparison (Logistic Regression, Decision Tree, Random Forest, XGBoost) on labelled flow-level data
- 📤 **Reproducible export** — all outputs (tables, metrics, figures) zipped for direct use in a report

> **Note:** PCAP forensic analysis alone does **not** provide ground-truth labels for supervised ML. The notebook strictly separates these two evaluation tracks.

---

## 🗺️ Pipeline

```
Dataset Download (HuggingFace)
        │
        ▼
Evidence Preservation (SHA-256 Hash + Working Copy)
        │
        ▼
PCAP Forensic Examination (TShark)
  ├── Protocol Summary
  ├── DNS Query Extraction
  ├── HTTP Request Extraction
  ├── Telnet Attempt Detection (ports 23 / 2323)
  ├── SYN / UDP / ICMP Source Counts
  └── Per-Second Traffic Statistics → forensic_timeline.csv
        │
        ▼
Optional Suricata IDS Pass (eve.json → alerts.csv)
        │
        ▼
Labelled CSV Preprocessing
  ├── Auto-detect label column (benign vs attack)
  ├── Handle missing / infinite values
  └── One-hot encode categorical features
        │
        ▼
ML Model Training & Evaluation
  ├── Logistic Regression
  ├── Decision Tree
  ├── Random Forest
  └── XGBoost (if installed)
        │
        ▼
Export → iot_botnet_case_study_results.zip
  ├── model_performance.csv
  ├── Confusion Matrix (PNG)
  ├── ROC Curve (PNG)
  ├── Feature Importance Plot (PNG)
  ├── dns_queries.csv / http_requests.csv / telnet_attempts.csv
  └── Suricata outputs (if run)
```

---

## 🚀 Quick Start

### Option 1 — Google Colab (Recommended)

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/YOUR_REPO/blob/main/IoT_Botnet_Forensics_and_ML.ipynb)

> Replace `YOUR_USERNAME/YOUR_REPO` with your actual GitHub repo path after uploading.

**Steps:**
1. Click **"Open in Colab"** above.
2. Go to **Runtime → Run all**, or run cells sequentially (recommended).
3. Cell 2 auto-downloads the Mirai PCAP + labelled CSV from Hugging Face — no manual upload needed.
4. After all cells complete, the final cell downloads `iot_botnet_case_study_results.zip` to your machine.

### Option 2 — Run Locally

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO

# 2. Install system dependencies (Ubuntu / Debian)
sudo apt-get install tshark suricata jq

# 3. Install Python dependencies
pip install pandas numpy scikit-learn matplotlib seaborn huggingface_hub

# 4. Launch Jupyter
jupyter notebook IoT_Botnet_Forensics_and_ML.ipynb
```

> ⚠️ Local runs require `tshark` and optionally `suricata` to be on your PATH. On macOS, install via `brew install wireshark suricata`.

---

## 📦 Dataset

| Property | Value |
|---|---|
| **Source** | [CICIoT2023Small — Hugging Face](https://huggingface.co/datasets/somnath0100/CICIoT2023Small) |
| **Attack Type** | Mirai UDP-Plain botnet |
| **Files Used** | `Mirai-udpplain.pcap` + `Mirai-udpplain.pcap_Flow.csv` |
| **Format** | PCAP for forensics · CSV (labelled flows) for ML |

Other compatible public datasets:

| Dataset | Organisation | Link |
|---|---|---|
| CICIoT2023 (full) | Canadian Institute for Cybersecurity | [Link](https://www.unb.ca/cic/datasets/iotdataset-2023.html) |
| Bot-IoT | UNSW Canberra | [Link](https://research.unsw.edu.au/projects/bot-iot-dataset) |
| N-BaIoT | UCI ML Repository | [Link](https://archive.ics.uci.edu/dataset/442/detection+of+iot+botnet+attacks+n+baiot) |

> For very large datasets, use a representative sample to stay within Colab RAM limits (~12 GB).

---

## 🛠️ Dependencies

| Tool | Purpose |
|---|---|
| `tshark` (Wireshark CLI) | PCAP parsing, protocol summaries, field extraction |
| `suricata` | Signature-based IDS alert generation |
| `capinfos` | PCAP metadata (bundled with Wireshark) |
| `pandas` | Data loading and preprocessing |
| `numpy` | Numerical operations |
| `scikit-learn` | ML models, metrics, confusion matrix |
| `matplotlib` / `seaborn` | ROC curve, confusion matrix, feature importance plots |
| `huggingface_hub` | Automatic dataset download from Hugging Face |
| `xgboost` *(optional)* | Gradient boosting classifier |

---

## 📁 Output File Structure

All outputs are written to `/content/forensic_outputs/` and packaged into a single ZIP at the end:

```
forensic_outputs/
├── evidence_copy/               # SHA-256-verified working copy of PCAP
├── protocol_summary.txt         # TShark I/O protocol hierarchy
├── dns_queries.csv              # Extracted DNS query records
├── http_requests.csv            # Extracted HTTP request records
├── telnet_attempts.csv          # Telnet traffic on ports 23 / 2323
├── forensic_timeline.csv        # Per-second packet rate timeline
├── source_ip_counts.csv         # Top SYN/UDP/ICMP flood sources
├── suricata/
│   ├── eve.json                 # Raw Suricata event log
│   ├── alerts.csv               # Parsed alert table
│   ├── run_stdout.txt
│   └── run_stderr.txt
├── model_performance.csv        # Accuracy, Precision, Recall, F1 per model
├── confusion_matrix.png         # Confusion matrix heatmap
├── roc_curve.png                # ROC curves (all models overlaid)
└── feature_importance.png       # Top-N feature importances (Random Forest)
```

---

## 🤖 Machine Learning Models

| Model | Notes |
|---|---|
| **Logistic Regression** | Linear baseline; interpretable coefficients, fast to train |
| **Decision Tree** | Fully interpretable; prone to overfit on raw flows |
| **Random Forest** | Ensemble; typically strongest generalisation on tabular network data |
| **XGBoost** | Gradient boosting; included if `xgboost` is pip-installed |

**Preprocessing pipeline:**
- Auto-detection of label column (benign vs. attack binary mapping)
- Drop or impute infinite / NaN values
- One-hot encode categorical features
- Stratified 80/20 train-test split

> ⚠️ Random splitting of near-duplicate flows from the same capture session may cause **data leakage**. A capture-based or device-based split is more rigorous when the dataset supports it.

---

## ⚠️ Limitations

| Limitation | Detail |
|---|---|
| **Offline data only** | No real-time packet capture or live hardware deployment |
| **Dataset-specific results** | Metrics reflect this specific capture; re-evaluate before generalising |
| **Possible label leakage** | Random splits on temporally correlated flows may inflate accuracy |
| **Encrypted payloads** | DPI (deep packet inspection) of encrypted traffic is out of scope |
| **Suricata rule coverage** | Default Colab Suricata may lack IoT-specific rules; zero alerts ≠ benign traffic |
| **Separate evaluation tracks** | Suricata alert rates and ML test-set metrics are **not directly comparable** |

---

## 📋 Case Study Reporting Checklist

- [ ] **Implementation details** — Colab runtime, Python version, dataset name/version, row count, label column, features retained, split ratio, tool versions
- [ ] **Model hyperparameters table** — document default parameters per model
- [ ] **Forensic indicators table** — populated from `dns_queries.csv`, `http_requests.csv`, `telnet_attempts.csv`, `forensic_timeline.csv`, and Suricata alerts; include evidence source and confidence; do not label an IP/domain malicious solely from its presence in the PCAP
- [ ] **Model performance table** — copy values **exactly** from `model_performance.csv`
- [ ] **Figures** — `confusion_matrix.png`, `roc_curve.png`, `feature_importance.png`
- [ ] **Limitations** — address all items above

---

## 📄 License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgements

- [Canadian Institute for Cybersecurity (CIC)](https://www.unb.ca/cic/) — CICIoT2023 dataset creators
- [Hugging Face](https://huggingface.co/datasets/somnath0100/CICIoT2023Small) — CICIoT2023Small hosted dataset
- [Suricata Open-Source IDS](https://suricata.io/) — network threat detection engine
- [Wireshark / TShark](https://www.wireshark.org/) — network protocol analyser
