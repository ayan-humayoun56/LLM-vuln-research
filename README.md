# LLM-Assisted Source Code Vulnerability Detection
### A Comparative Study of Prompting Strategies on Open-Source Models

<div align="center">

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=flat-square)
![Model](https://img.shields.io/badge/Model-CodeLlama--7B-blue?style=flat-square)
![Model](https://img.shields.io/badge/Model-DeepSeek--Coder--6.7B-blue?style=flat-square)
![Dataset](https://img.shields.io/badge/Dataset-CVEfixes-green?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square)
![Target](https://img.shields.io/badge/Target%20Venue-IEEE%20Access-red?style=flat-square)


</div>

---

## Overview

This repository contains the complete code, data pipeline, prompts, and evaluation scripts for our undergraduate thesis research on **evaluating LLM prompting strategies for source code vulnerability detection**.

We systematically compare **4 prompting strategies** across **2 open-source large language models** (CodeLlama-7B-Instruct and DeepSeek-Coder-6.7B-Instruct) against **3 traditional static analysis tools** (Semgrep, Bandit, Flawfinder) on a balanced dataset of 1,000 real-world vulnerable and safe code samples drawn from the CVEfixes dataset.

Our primary research question:

> *"How effectively do different LLM prompting strategies detect real-world source code vulnerabilities, and how do they compare to traditional static analysis baselines?"*

---

## Key Contributions

- **Contribution 1** : First head-to-head comparison of 4 prompting strategies (zero-shot, few-shot, chain-of-thought, CWE-guided) across 2 open-source LLMs on the same labeled dataset
- **Contribution 2** : Quantitative comparison against 3 established static analysis tools using precision, recall, F1-score, and false positive rate
- **Contribution 3** : A structured taxonomy of LLM failure modes categorized by CWE vulnerability type, grounded in manual security analysis

---

## Experiment Matrix

| Prompting Strategy | CodeLlama-7B-Instruct | DeepSeek-Coder-6.7B-Instruct |
|---|:---:|:---:|
| Zero-shot | ✅ | ✅ |
| Few-shot (3 examples) | ✅ | ✅ |
| Chain-of-Thought | ✅ | ✅ |
| CWE-Guided *(novel)* | ✅ | ✅ |

Plus baselines: **Semgrep** · **Bandit** · **Flawfinder**

**Total: 11 experimental configurations evaluated on 1,000 code samples.**

---

## Repository Structure
```bash
llm-vuln-research/
│
├── data/
│   ├── raw/                    # Original downloaded datasets (not tracked by git)
│   ├── processed/
│   │   ├── dataset.csv         # 1,000 balanced samples (500 vuln / 500 safe)
│   │   └── train_test_split/   # 70/30 split for any future fine-tuning
│   └── results/
│       ├── semgrep.json        # Semgrep raw output
│       ├── bandit.json         # Bandit raw output
│       ├── flawfinder.csv      # Flawfinder raw output
│       └── combined_results.csv  # All 11 prediction columns merged
│
├── scripts/
│   ├── build_dataset.py        # Download, filter, balance CVEfixes to 1,000 samples
│   ├── export_samples.py       # Export code snippets as individual .py / .c files
│   ├── run_static_tools.sh     # Shell script to run Semgrep, Bandit, Flawfinder
│   ├── parse_tool_outputs.py   # Parse JSON/CSV tool outputs → dataset.csv columns
│   ├── run_llm.py              # Main LLM inference script (HuggingFace Inference API)
│   ├── all_metrics.py          # Compute precision / recall / F1 for all 11 configs
│   └── extract_errors.py       # Extract false negatives and false positives for analysis
│
├── prompts/
│   ├── zero_shot.txt           # Prompt strategy 1
│   ├── few_shot.txt            # Prompt strategy 2
│   ├── chain_of_thought.txt    # Prompt strategy 3
│   └── cwe_guided.txt          # Prompt strategy 4 (novel CWE-aware prompting)
│
├── errors/
│   ├── false_negatives.csv     # Missed vulnerabilities — annotated with failure reason
│   ├── false_positives.csv     # Incorrect alerts — annotated with failure reason
│   └── error_taxonomy.xlsx     # Structured taxonomy of LLM failure modes by CWE type
│
├── notebooks/
│   ├── dataset_exploration.ipynb   # EDA on CVEfixes dataset
│   └── results_visualization.ipynb # Generate all paper figures
│
├── paper/                      # Overleaf-synced LaTeX source (available after submission)
│
├── notes/
│   ├── meeting_notes/          # Weekly supervisor meeting notes
│   └── literature_notes.md     # Summary notes on papers read
│
├── requirements.txt
├── .env.example                # Template for environment variables (HF token, etc.)
├── .gitignore
└── README.md
```
---

## Dataset

We use the **[CVEfixes dataset](https://github.com/secureIT-project/CVEfixes)** (Bhandari et al., 2021), a curated collection of real-world CVE patches with before/after code pairs.

| Property | Value |
|---|---|
| Source | CVEfixes (Zenodo DOI: 10.5281/zenodo.7029359) |
| Languages | Python, C, C++ |
| Total samples | 1,000 (balanced) |
| Vulnerable samples | 500 |
| Safe samples | 500 |
| Max function length | 200 lines |
| CWE types covered | CWE-89, CWE-79, CWE-120/121, CWE-416, CWE-476, CWE-22 |

> **Note:** The raw dataset is not stored in this repository due to size. See [Setup](#quick-start) for download instructions. The processed `dataset.csv` with ground truth labels is included.

---

## Models

| Model | Parameters | Source | Why chosen |
|---|---|---|---|
| `CodeLlama-7b-Instruct-hf` | 7B | Meta AI via HuggingFace | Code-specialized, widely cited in security research, reproducible |
| `deepseek-coder-6.7b-instruct` | 6.7B | DeepSeek AI via HuggingFace | Strong code benchmark performance, open-source, 2024 release |

Both models are accessed via the **HuggingFace Inference API** — no local GPU required.

---

## Quick Start

### Prerequisites

```bash
git clone https://github.com/YOUR_USERNAME/llm-vuln-research.git
cd llm-vuln-research
pip install -r requirements.txt
```

### Environment Setup

```bash
cp .env.example .env
# Add your HuggingFace token to .env:
# HF_TOKEN=hf_your_token_here
```

Get a free HuggingFace token at [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens).

### 1. Build the Dataset

```bash
python scripts/build_dataset.py
# Output: data/processed/dataset.csv (1,000 samples)
```

### 2. Run Static Analysis Baselines

```bash
# Export code samples as individual files first
python scripts/export_samples.py

# Run all three tools
bash scripts/run_static_tools.sh

# Parse outputs into dataset.csv
python scripts/parse_tool_outputs.py
```

### 3. Run LLM Experiments

```bash
# Usage: python scripts/run_llm.py <model> <strategy>
# Models:     codellama | deepseek
# Strategies: zero_shot | few_shot | cot | cwe_guided

python scripts/run_llm.py codellama zero_shot
python scripts/run_llm.py codellama few_shot
python scripts/run_llm.py codellama cot
python scripts/run_llm.py codellama cwe_guided

python scripts/run_llm.py deepseek zero_shot
python scripts/run_llm.py deepseek few_shot
python scripts/run_llm.py deepseek cot
python scripts/run_llm.py deepseek cwe_guided
```

> Results are saved incrementally to `data/processed/dataset.csv` every 50 samples. If a run is interrupted, re-running the same command will resume from where it left off.

### 4. Compute All Metrics

```bash
python scripts/all_metrics.py
# Prints precision, recall, F1, FPR for all 11 configurations
```

---

## Prompting Strategies

<details>
<summary><strong>Strategy 1 — Zero-Shot</strong></summary>
You are a security expert. Analyze the following source code for security vulnerabilities.
Respond with ONLY one word: VULNERABLE or SAFE
Code:
{code}
</details>

<details>
<summary><strong>Strategy 2 — Few-Shot (3 examples)</strong></summary>
You are a security expert. Here are labeled examples:
EXAMPLE 1 - VULNERABLE:
query = "SELECT * FROM users WHERE id=" + user_id
(Reason: SQL injection — unsanitized user input in query)
EXAMPLE 2 - SAFE:
cursor.execute("SELECT * FROM users WHERE id=?", (user_id,))
(Reason: Parameterized query — injection not possible)
EXAMPLE 3 - VULNERABLE:
strcpy(buf, input);
(Reason: No bounds check — buffer overflow)
Now analyze:
{code}
Respond ONLY: VULNERABLE or SAFE
</details>

<details>
<summary><strong>Strategy 3 — Chain-of-Thought</strong></summary>
You are a security code reviewer. Think step by step:

What are the external inputs?
Where do inputs flow through the code?
Do any inputs reach dangerous operations without sanitization?
State your verdict.

Code:
{code}
End your response with exactly: VERDICT: VULNERABLE or VERDICT: SAFE
</details>

<details>
<summary><strong>Strategy 4 — CWE-Guided (Novel Contribution)</strong></summary>
You are a senior application security engineer. Check this code specifically for:

CWE-89: SQL Injection
CWE-79: Cross-site Scripting
CWE-120/121: Buffer Overflow
CWE-416: Use-After-Free
CWE-476: NULL Pointer Dereference
CWE-22: Path Traversal

Code:
{code}
Respond with ONLY: VULNERABLE or SAFE
</details>

---

## Evaluation Metrics

| Metric | Definition |
|---|---|
| **Precision** | Of all predicted VULNERABLE, how many were actually vulnerable? |
| **Recall** | Of all actual vulnerabilities, how many did we catch? |
| **F1-Score** | Harmonic mean of precision and recall. Primary headline metric. |
| **False Positive Rate (FPR)** | Of all safe samples, what fraction was incorrectly flagged? |

Results are reported per-tool and per-CWE-type. All experiments use `random_state=42` for reproducibility.

---

## Error Taxonomy

A key contribution of this work is the manual analysis of LLM failure cases. We annotated 80+ false negatives and false positives and grouped them into the following failure categories:

| Category | Description |
|---|---|
| **Inter-procedural** | Vulnerability spans multiple functions; LLM sees only one at a time |
| **Memory-model dependent** | Use-after-free, dangling pointers — requires reasoning about heap state |
| **Context-dependent FP** | LLM flags a dangerous-looking pattern that is safe due to prior validation |
| **Multi-step chain** | Vulnerability requires chaining two weaknesses (e.g., overflow → injection) |
| **Language idiom** | Language-specific pattern misinterpreted (C pointer arithmetic, Python f-strings) |

The full annotated dataset is in `errors/error_taxonomy.xlsx`.

---

## Requirements
pandas>=2.0.0
scikit-learn>=1.3.0
huggingface_hub>=0.20.0
transformers>=4.35.0
semgrep
bandit
flawfinder
matplotlib>=3.7.0
seaborn>=0.12.0
tqdm>=4.65.0
python-dotenv>=1.0.0

Install all:
```bash
pip install -r requirements.txt
```

---

## Reproducibility

- **Fixed random seed:** All dataset splitting uses `random_state=42`
- **Open-source models only:** Both models are publicly available on HuggingFace — no proprietary APIs required
- **Deterministic inference:** All LLM calls use `temperature=0.01`
- **Code and prompts versioned:** All prompt files and scripts are in this repository
- **Dataset publicly available:** CVEfixes is archived on Zenodo with a permanent DOI

---

## Paper

> **Status:** Under preparation — targeting IEEE Access

**Title:** LLM-Assisted Source Code Vulnerability Detection: A Comparative Study of Prompting Strategies

**Authors:** Ayan Humayoun, Syed Shabahat Hussain

**Abstract:** *(coming soon — will be updated on arXiv submission)*

**Preprint:** *(arXiv link will be added on submission)*

**Target venue:** [IEEE Access](https://ieeeaccess.ieee.org/) (Open Access, Impact Factor ~3.4)

---

## Citation

```bibtex
@article{humayoun2025llmvuln,
  title     = {LLM-Assisted Source Code Vulnerability Detection: A Comparative Study of Prompting Strategies},
  author    = {Ayan Humayoun and Syed Shabahat Hussain},
  journal   = {IEEE Access},
  year      = {2025},
  note      = {Under review. Preprint: arXiv:2025.XXXXX}
}
```

---

## Acknowledgements

We thank our thesis supervisor for guidance on research methodology and paper writing. This research uses the publicly available [CVEfixes dataset](https://github.com/secureIT-project/CVEfixes) by Bhandari et al. (2021) and open-source models from Meta AI (CodeLlama) and DeepSeek AI via the HuggingFace Hub.

---

## Contact

| Researcher | Role | GitHub |
|---|---|---|
| **Ayan Humayoun** | Dataset, Metrics, Results | [@ayan-humayoun](https://github.com/ayan-humayoun56) |
| **Syed Shabahat Hussain** | LLM Experiments, Error Analysis | [@shabahat-hussain](https://github.com/shabahat1) |

For questions about this research, open a GitHub Issue or reach out via the university email.

---

<div align="center">

*BSCS Undergraduate Thesis · Department of Computer Science*

![Made with Python](https://img.shields.io/badge/Made%20with-Python-blue?style=flat-square&logo=python)
![HuggingFace](https://img.shields.io/badge/Models-HuggingFace-yellow?style=flat-square)
![CVEfixes](https://img.shields.io/badge/Dataset-CVEfixes-green?style=flat-square)

</div>
