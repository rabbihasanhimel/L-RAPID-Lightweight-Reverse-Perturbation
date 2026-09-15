# L-RAPID: Lightweight Reversible Adversarial Perturbation for Industrial Modbus Data

[![Conference](https://img.shields.io/badge/Conference-IEEE%20CyberMACS%202026-blue.svg)](paper/lrapid_camera_ready.pdf)
[![Paper Status](https://img.shields.io/badge/Status-Camera--Ready%20Accepted-success.svg)](paper/lrapid_camera_ready.pdf)
[![Language](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Security](https://img.shields.io/badge/Security-Dual--Adversary%20Evaluated-orange.svg)](docs/L_RAPID_Defense_and_Interview_Notes.docx)

---

## 📌 Executive Summary

Modern Industrial Control Systems (ICS) and SCADA environments rely heavily on legacy protocols such as **Modbus TCP/IP**. Because Modbus transmits operational telemetry in cleartext without built-in encryption, critical sensor/actuator registers are vulnerable to eavesdropping, reconnaissance, and unauthorized machine-learning-driven process inference.

Conventional security approaches face severe bottlenecks in industrial deployments:
- **Standard Cryptography (AES-128-CBC / CTR, ChaCha20):** Introduces non-deterministic packet overhead, byte expansion, and microsecond latencies (6–12 µs/packet) that violate hard real-time SCADA cycle limits (10–50 ms) on memory-constrained 8/16-bit PLCs and RTUs.
- **Differential Privacy (Gaussian / Laplace AGP):** Adds irreversible statistical noise that destroys floating-point data fidelity, causing false alarms in downstream anomaly detectors and expanding 16-bit integer registers by 300% into 64-bit IEEE floats.

**L-RAPID** resolves this dilemma by introducing a **deterministic, sub-microsecond, reversible bitwise scrambling mechanism** engineered specifically for 16-bit Modbus industrial registers. By packaging multi-register telemetry into 64-bit SIMD words, L-RAPID delivers:
- **Sub-microsecond latency:** **0.11 µs/packet** (XOR-64 SIMD) and **0.70 µs/packet** (XOR-16 native).
- **Extreme speedup:** **46.2× faster than ChaCha20** and **64.0× faster than AES-CBC**.
- **100% Bit-Exact Reversibility:** Zero residual numerical error ($\Delta = 0$), preserving 100% analytic utility for authorized cloud SCADA engines.
- **Zero Wire Overhead:** 0.0% packet length expansion (keeps native Modbus PDU compliance).

---

## 🏗️ Architecture & Operational Flow

```
[ Field Sensor / RTU ]
       │  (Plaintext Modbus Holding/Input Registers: 4 × 16-bit uint16)
       ▼
┌────────────────────────────────────────────────────────┐
│               L-RAPID Edge Perturbation                │
│  Keystream Gen: K_t = LCG_PRNG(Seed = S_0, Counter = t)│
│  Perturbation : C_t = P_t ⊕ (K_t & Mask_k)            │
│  SIMD Mode    : Pack 4 × uint16 → uint64 ⊕ K_64        │
└────────────────────────────────────────────────────────┘
       │  Execution Latency: 0.11 µs (SIMD) / 0.70 µs (Native)
       │  Payload Overhead: 0.0% (8 bytes in, 8 bytes out)
       ▼
[ Untrusted Industrial Network / Telemetry Wiretap ]
       │  Adversary Observation: High Entropy, Uniformly Scrambled Data
       │  Attacker ML Classification Accuracy: Collapses to Random Baseline (78.9% binary / 50% guessing)
       ▼
┌────────────────────────────────────────────────────────┐
│             Authorized Cloud / SCADA Master            │
│  Reversal     : P_t = C_t ⊕ (K_t & Mask_k)             │
│  Restoration  : Δ = |P_restored - P_original| = 0      │
└────────────────────────────────────────────────────────┘
       │
       ▼
[ High-Precision Downstream ML & Safety Analytics ]
  Analytical Accuracy: 99.85% (100% Identical to Clean Baseline)
```

---

## 📊 Key Benchmark & Experimental Results

Evaluated on **31,106 real-world Modbus network telemetry records** (BoT-IoT / Tonekaboni et al.):

### 1. Computational & Latency Benchmark Comparison

| Protection Method | Execution Paradigm | Enc. Latency (µs/sample) | Per-Packet Latency (µs) | Real-Time SCADA Speedup | Wire Overhead (%) | Exact Match Reversibility (%) |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **L-RAPID (XOR-64 SIMD)** | SIMD Packed Vector | **0.0008 µs** | **0.119 µs** | **Baseline (1.0×)** | **0.0%** | **100.0% (Exact)** |
| **L-RAPID (XOR-16 Native)** | Register-Level | **0.0067 µs** | **0.714 µs** | **6.0× slower** | **0.0%** | **100.0% (Exact)** |
| **ChaCha20 Stream Cipher** | 256-bit CSPRNG Stream | 0.0414 µs | 5.402 µs | 45.3× slower | 0.0% | 100.0% (Exact) |
| **AES-128-CTR Mode** | Stream Block Cipher | 0.0051 µs | 6.624 µs | 55.6× slower | 0.0% | 100.0% (Exact) |
| **AES-128-CBC Mode** | Block + PKCS7 Pad | 0.0102 µs | 7.399 µs | 62.1× slower | 0.0% | 100.0% (Exact) |
| **Additive Gaussian (AGP)** | Differential Privacy | 0.0125 µs | 1.520 µs | 12.8× slower | **+300.0% (Float64)** | **0.0% (Lossy)** |

### 2. Dual-Adversary Machine Learning Security Evaluation

| Operational Scenario | Protected Feature State | Binary Attack Acc (%) | Binary F1 | Multi-Class Acc (%) | Operational Threat / Security Outcome |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Clean Baseline** | Raw Unperturbed | 99.85% | 0.9991 | 99.82% | Unprotected cleartext: full reconnaissance threat |
| **Zero-Knowledge Wiretap** | L-RAPID Encrypted ($k=16$) | **78.93%** | **0.8822** | **78.93%** | **Attacker blind: feature distribution scrambled** |
| **Transfer Attack (IDS)** | Pre-trained Model on Encrypted | **46.27%** | **0.5878** | **23.41%** | **Model collapse: worse than random coin-flip** |
| **Authorized Cloud Master** | L-RAPID Restored ($\Delta = 0$) | **99.85%** | **0.9991** | **99.82%** | **Zero utility loss: 100% analytics preserved** |

---

## 📁 Repository Structure

```text
.
├── README.md                          <- Repository documentation, benchmarks, and quickstart
├── .gitignore                         <- Comprehensive exclusions for Python, Jupyter, LaTeX, Office
├── LICENSE                            <- MIT Open-Source License
│
├── modbus(latest)/                    <- Core experimental and benchmarking workspace
│   ├── Train_Test_IoT_Modbus.csv      <- BoT-IoT Modbus network telemetry (31,106 records)
│   ├── benchmark_crypto_comparison.py <- Microsecond execution benchmark (L-RAPID vs AES vs ChaCha20)
│   ├── parametric_bitdepth_analysis.py<- Bit depth k ∈ {1..16} tradeoff analysis
│   ├── run_dual_adversary_eval.py     <- Dual-Adversary ML evaluation (Static vs Adaptive)
│   ├── benchmark_results.csv          <- Micro-benchmark metrics
│   ├── dual_adversary_summary.csv     <- Dual-adversary ML evaluation summary
│   ├── parametric_k_results.csv       <- Parametric bit-depth evaluation table
│   ├── tradeoff_k_vs_accuracy.png     <- Privacy-utility tradeoff curve
│   ├── confusion_matrices_triad.png   <- Triad confusion matrices (Clean, Perturbed, Recovered)
│   ├── nb1_*.ipynb                    <- XOR reversible perturbation notebooks (16-bit, 64-bit SIMD)
│   ├── nb2_*.ipynb                    <- Gaussian perturbation comparison notebooks
│   ├── nb3_*.ipynb                    <- Downstream machine learning analysis pipelines
│   └── data_*_encrypted/decrypted.csv <- Intermediate perturbed and decrypted datasets
│
├── paper/                             <- Camera-ready conference submission & literature
│   ├── lrapid.tex                     <- Complete IEEE conference LaTeX source (Table I, diacritics)
│   ├── lrapid_camera_ready.pdf        <- Verified 2-page camera-ready PDF
│   ├── references/                    <- 9 foundational reference research papers (PDF)
│   └── archive/                       <- Prior drafts and backup builds
│
├── presentation/                      <- Conference slide deck & presentation assets
│   ├── CyberMACS_Conference_Presentation_LRAPID.pptx  <- Modern academic light-theme deck (13 slides)
│   └── archive/                       <- Previous iteration drafts and user backup
│
└── docs/                              <- Defense notes, project briefs, and technical drafts
    ├── L_RAPID_Defense_and_Interview_Notes.docx  <- Defense guide, reviewer Q&A, ChaCha20 analysis
    ├── Draft_Paper_RDP_Modbus.md      <- 10-page expanded journal paper draft
    ├── Dataset_Analysis_FINAL.docx    <- Exploratory data analysis report
    ├── Project_Updates_for_Team.docx  <- Project updates for research team
    ├── CASE_III_Challenge_Brief.pdf   <- Original challenge specification
    ├── purpose.pdf                    <- Project scope document
    ├── limitation_and_updates.docx    <- Limitations & notes
    └── assets/                        <- Supplementary images & submission receipt
```

---

## 🚀 Quickstart & Reproduction Guide

### 1. Prerequisites & Environment Setup

Clone this repository and ensure Python 3.10+ is installed:

```bash
# Clone the repository
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>

# (Optional) Create and activate a virtual environment
python -m venv venv
# On Windows PowerShell:
.\venv\Scripts\Activate.ps1
# On Linux/macOS:
source venv/bin/activate

# Install required dependencies
pip install pandas numpy scikit-learn matplotlib seaborn cryptography
```

### 2. Running Micro-Benchmarks

To benchmark L-RAPID against ChaCha20, AES-128-CTR, AES-128-CBC, and Gaussian Perturbation:

```bash
cd "modbus(latest)"
python benchmark_crypto_comparison.py
```
*Outputs are saved to `modbus(latest)/benchmark_results.csv`.*

### 3. Running Dual-Adversary ML Evaluation

To evaluate classification accuracy under clean, wiretapped, transfer attack, and restored scenarios:

```bash
cd "modbus(latest)"
python run_dual_adversary_eval.py
```
*Outputs are saved to `modbus(latest)/dual_adversary_summary.csv` and `modbus(latest)/confusion_matrices_triad.png`.*

### 4. Running Parametric Bit-Depth Analysis ($k = 1 \dots 16$)

To generate the privacy-utility tradeoff curve across varying perturbation bit depths:

```bash
cd "modbus(latest)"
python parametric_bitdepth_analysis.py
```
*Outputs are saved to `modbus(latest)/parametric_k_results.csv` and `modbus(latest)/tradeoff_k_vs_accuracy.png`.*

---

## 📑 Paper & Conference Details

- **Conference:** CyberMACS International Applied Cybersecurity Conference & Summer School 2026
- **Paper Title:** L-RAPID: Lightweight Reversible Adversarial Perturbation for Industrial Modbus Data
- **Camera-Ready PDF:** [`paper/lrapid_camera_ready.pdf`](paper/lrapid_camera_ready.pdf)
- **LaTeX Source:** [`paper/lrapid.tex`](paper/lrapid.tex)
- **Presentation Deck:** [`presentation/CyberMACS_Conference_Presentation_LRAPID.pptx`](presentation/CyberMACS_Conference_Presentation_LRAPID.pptx)

### Citation (BibTeX)

```bibtex
@inproceedings{balli2026lrapid,
  author    = {Tu{\u{g}}{\c{c}}e Ball{\i} and Rabbi Hasan Himel and Contributors},
  title     = {L-RAPID: Lightweight Reversible Adversarial Perturbation for Industrial Modbus Data},
  booktitle = {Proceedings of the CyberMACS International Applied Cybersecurity Conference \& Summer School},
  year      = {2026},
  publisher = {IEEE / TÜBİTAK BİLGEM},
  address   = {Istanbul, Turkey}
}
```

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.
