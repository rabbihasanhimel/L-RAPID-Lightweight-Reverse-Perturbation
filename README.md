# L-RAPID: Lightweight Reversible Adversarial Perturbation for Industrial Modbus Data

## Abstract

Industrial Control Systems (ICS) using legacy protocols like Modbus TCP/IP transmit operational telemetry across networks in cleartext, making them vulnerable to eavesdropping, reconnaissance, and unauthorized machine-learning-driven process inference. Standard cryptographic encryption (such as AES or ChaCha20) introduces processing latency and protocol overhead that violate strict real-time SCADA cycle constraints. Alternatively, Differential Privacy techniques add irreversible numerical noise, degrading downstream analytical accuracy and expanding register payload sizes.

L-RAPID addresses this challenge by applying lightweight, reversible bitwise keystream scrambling directly to Modbus register telemetry. By packing multiple 16-bit registers into 64-bit words, L-RAPID achieves sub-microsecond latency (0.11 µs for 64-bit SIMD, 0.71 µs for 16-bit native) with zero packet size expansion. At the authorized cloud or control center, the process is 100% bit-exact reversible, preserving complete analytic utility for downstream safety and anomaly detection systems while collapsing eavesdropper classification accuracy to random guessing.

---

## Repository Structure

- `modbus(latest)/`: Core codebase containing all Python scripts, Jupyter notebooks, datasets, and generated benchmark figures.
  - `Train_Test_IoT_Modbus.csv`: Dataset containing 31,106 Modbus network records.
  - `benchmark_crypto_comparison.py`: Execution time benchmark comparing L-RAPID against AES-128-CTR, AES-128-CBC, ChaCha20, and Gaussian perturbation.
  - `run_dual_adversary_eval.py`: Machine learning security evaluation (Clean baseline vs. Wiretap vs. Restored).
  - `parametric_bitdepth_analysis.py`: Privacy-utility tradeoff evaluation across perturbation bit depths (k = 1 ... 16).
  - `nb1_*.ipynb` to `nb3_*.ipynb`: Step-by-step exploratory and evaluation Jupyter notebooks.
- `paper/`: Conference paper materials.
  - `lrapid.tex`: IEEE conference paper LaTeX source.
  - `lrapid_camera_ready.pdf`: 2-page camera-ready accepted paper.
  - `references/`: Reference literature and related research papers.
- `presentation/`: Conference slides for CyberMACS 2026.
  - `CyberMACS_Conference_Presentation_LRAPID.pptx`: 13-slide academic presentation.
- `docs/`: Technical documentation, project notes, defense/interview preparation, and extended drafts.

---

## Requirements & Setup

Install the required Python packages:

```bash
pip install pandas numpy scikit-learn matplotlib seaborn cryptography


cd "modbus(latest)"

# 1. Run cryptographic latency benchmarks
python benchmark_crypto_comparison.py

# 2. Run dual-adversary machine learning evaluation
python run_dual_adversary_eval.py

# 3. Run parametric bit-depth analysis
python parametric_bitdepth_analysis.py

```

---

## Conference & Citation

Accepted for presentation at the **CyberMACS International Applied Cybersecurity Conference & Summer School (2026)**.

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

## License

This project is released under the [MIT License](LICENSE).

