# 🧬 BKV VP1 Subtyping Pipeline

> **De novo sub-lineage identification for BK polyomavirus VP1 fragments**

![Python](https://img.shields.io/badge/Python-3.12+-blue?logo=python&logoColor=white)
![Biopython](https://img.shields.io/badge/Biopython-1.86+-green?logo=biopython&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20macOS%20%7C%20Jupyter-orange)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Version Report](#-version-report)
- [Pipeline Steps](#-pipeline-steps)
- [Reference Panel](#-reference-panel)
- [Installation](#-installation)
- [Usage](#-usage)
- [Output Files](#-output-files)
- [Key Findings](#-key-findings)
- [Limitations](#-limitations)
- [Citation](#-citation)

---

## 📖 Overview

This pipeline performs **de novo subtyping** of BK polyomavirus (BKPyV) VP1 capsid gene fragments without relying on NCBI annotation tags. It uses a comprehensive reference panel of **112 explicitly genotype-tagged VP1 sequences** covering all major sub-lineages (Ia, Ib1, Ib2, Ic, II, III, IV) to classify query sequences via:

- 🌳 Neighbor-Joining phylogenetic tree
- 📊 Pairwise distance matrix (p-distance)
- 🔬 Subtype-diagnostic residue scan (protein + nucleotide level)
- 🧪 Novel substitution detection
- 📈 Confidence tier assignment (HIGH / MODERATE / LOW)

---

## 📌 Version Report

| Component | Version | Date | Status |
|-----------|---------|------|--------|
| **Pipeline** | `v2.0` | 2026-10-23 | ✅ Stable |
| **Notebook** | `v2.0` | 2026-10-23 | ✅ Bug-fixed |
| **Reference Panel** | `v3.0` | 2026-10-23 | ✅ 112 sequences |
| **CSV Export** | `v2.0` | 2026-10-23 | ✅ 4-column format |
| **Agent Prompt** | `v1.0` | 2026-10-23 | ✅ GLM-tuned |

### 🐛 Bug History

| # | Bug | Fix | Version |
|---|-----|-----|---------|
| 1 | `NewickIO.to_newick()` does not exist in Biopython | Replaced with `NewickIO.write([tree], buf)` + `buf.getvalue()` | v2.0 |
| 2 | `/note="BKPyVgp4"` misinterpreted as subtype IV | Discovered `/note="BKPyVgpN"` is a gene-product index, not subtype | v2.0 |
| 3 | VP1 nt length filter rejected 1089 bp (with stop codon) | Extended filter to accept 1086, 1089, 1092 | v2.0 |
| 4 | Ambiguous IUPAC codes (N/M/R) inflated fingerprint counts | Added `VALID_NTS = set("ATCG")` filter | v2.0 |

---

## 🔄 Pipeline Steps

| Step | Description | Tool |
|------|-------------|------|
| 0️⃣ | **Input QC & Translation** — verify length, GC, translate 3 frames, pick cleanest ORF | Biopython `Seq.translate()` |
| 1️⃣ | **BLASTn vs NCBI nr/nt** — submit query, poll, retrieve top 50 hits | NCBI qblast URL API |
| 2️⃣ | **Load reference panel** — parse 112 `.gb` files, extract VP1 CDS + genotype tag | Biopython `SeqIO` |
| 3️⃣ | **Multiple sequence alignment** — column-wise (all VP1 = 362 aa, no indels) | Biopython `MultipleSeqAlignment` |
| 4️⃣ | **NJ phylogenetic tree** — p-distance, Neighbor-Joining | Biopython `DistanceTreeConstructor` |
| 5️⃣ | **Pairwise distance matrix** — per-sub-lineage match counts | Python sets |
| 6️⃣ | **Subtype-diagnostic scan** — variable positions, SYN/NON-SYN, novel detection | Python + Biopython |
| 7️⃣ | **Recombination test** — skipped if fragment < 500 bp | RDP5/GARD (if available) |
| 8️⃣ | **Final call** — reconcile signals, assign confidence tier | Decision rules |

---

## 📚 Reference Panel

| Sub-lineage | Count | Key Prototypes |
|-------------|-------|----------------|
| **Ia** | 15 | Dunlop (V01108.1), Cub_JA (KF740811.1) |
| **Ib1** | 22 | HSCT23 (PV856439), KTR12 (PV856423) |
| **Ib2** | 35 | HSCT24 (PV856440), **PV856451** (rare V-form!) |
| **Ic** | 18 | 25B40343 (PZ129847), 24B48251 (PZ129825) |
| **II** | 5 | P17_Plasma_BKVN (PX257722) |
| **III** | 5 | VP1M21 (JX195577), KT-514-USA (KF055891) |
| **IV** | 6 | HSCT05 (PV856455) — canonical IV reference |
| **I** (general) | 6 | VP1M14-M20 (JX195570-576) |
| **Total** | **112** | All with explicit `/note="genotype: Xn"` tags |

---

## ⚙️ Installation

```bash
pip install biopython>=1.86
```

### Requirements

- 🐍 Python 3.12+
- 🧬 Biopython 1.86+
- 🌐 Internet access (for NCBI BLAST + EUtils)
- 📁 Reference panel in `/home/z/my-project/download/references/*.gb`

---

## 🚀 Usage

### Option A: Jupyter Notebook

1. Open `bkv_vp1_subtyping_pipeline.ipynb` in Jupyter / VS Code / Colab
2. Replace the `QUERY` variable in Cell 1 with your sequence
3. Run all cells (Shift+Enter)

### Option B: Python Script

```python
from pathlib import Path
from Bio.Seq import Seq

QUERY = "GGTAGATGCTATTACAGAGGTAGAATGCTTCCTAAACCCAGAAATGGGGGATCCAGATGAAAACCTTAGG..."
# ... rest of pipeline
```

### Option C: GLM Agent Prompt

Use `agent_prompt_glm.txt` — paste it as the first message to a GLM model, then provide your sequence.

---

## 📁 Output Files

| File | Description |
|------|-------------|
| `outputs/query.fasta` | Input query (FASTA) |
| `outputs/query_translated.fasta` | Translated protein (FASTA) |
| `outputs/blast_report.txt` | Raw BLAST text report |
| `outputs/tree.newick` | NJ tree (Newick format) |
| `reference_sequences_trimmed.csv` | Reference panel with fingerprints |
| `final_report_*.txt` | Comprehensive analysis report |

### CSV Format (4 columns)

| Column | Header | Example |
|--------|--------|---------|
| 1 | `trimmed_reference_sequence` | `GTAGATGCTATTACAGAGG...` (249 bp) |
| 2 | `subtype` | `Ia` |
| 3 | `unique_subtype_fingerprints_count_out_of_total` | `3/12` |
| 4 | `accession_number` | `JF894228.1` |

---

## 🎯 Key Findings

### The 5 "Sub-lineage Switch" Positions

These 5 positions in the 249-bp fragment determine the sub-lineage call:

| Position | VP1 Residue | Residue Options | Effect |
|----------|-------------|-----------------|--------|
| **12** | 45 | T → Ia-specific | Rules out Ib1/Ib2/Ic |
| **123** | 82 | G → Ia-G-form | A → Ib1/Ib2 form |
| **165** | 96 | G → Ia-G-form ONLY | C → Ia-C-form or Ib1/Ib2/Ic |
| **222** | 115 | A → A-form Ib2 ONLY | T → Ia/Ib1/Ic |
| **237** | 120 | C → Ib1/Ib2 form | T → Ia/Ic form |

### Call Decision Matrix

| Position 123 | Position 165 | Position 222 | Position 237 | nt-level Best | Call | Confidence |
|---|---|---|---|---|---|---|
| G | G | T | T | Ia 100% | **Ia** | **HIGH** |
| A | C | T | T | Ia 92/93 | mosaic | MODERATE |
| A | C | T | C | Ib1=Ib2 92/93 | E-form Ib1 | MODERATE |
| A | C | A | C | Ib2 92/93 | A-form Ib2 | MODERATE |

> ⚠️ Position 12 (T) is **always Ia-specific** — it's the "Ia anchor" that prevents HIGH-confidence Ib1/Ib2 calls.

---

## ⚠️ Limitations

1. **Fragment size**: 249 bp is below the recommended minimum (500 bp) for formal recombination testing (RDP5/GARD)
2. **Sub-lineage resolution**: The 83-aa fragment cannot distinguish Ia from E-form Ib1 from Ic at the protein level — they share identical residues
3. **Single-position dependency**: The call can change based on a single synonymous substitution at positions 123, 165, 222, or 237
4. **Reference panel bias**: Ib1 has more E-form references (19/22) than Ia (9/15), inflating Ib1's protein-level match count
5. **Novel substitutions**: If the query carries residues not seen in any reference (like K69H or E73A), the protein-level match drops to 0% and the call becomes UNCLASSIFIABLE

### 🔬 For Definitive Resolution

Sequence the **full VP1** (~1086 bp), especially the **HI loop** (aa 225-300), which carries the most sub-lineage-discriminative residues.

---

## 📜 Citation

If you use this pipeline, please cite:

```
BKV VP1 Subtyping Pipeline v2.0 (2026)
De novo sub-lineage identification from partial VP1 fragments
Reference panel: 112 genotype-tagged VP1 sequences (Ia=15, Ib1=22, Ib2=35, Ic=18, II=5, III=5, IV=6)
```

---

## 📄 License

MIT License — see [LICENSE](LICENSE) file for details.

---

## 🔗 Links

- [NCBI BLAST](https://blast.ncbi.nlm.nih.gov/Blast.cgi)
- [NCBI EUtils](https://eutils.ncbi.nlm.nih.gov/entrez/eutils/)
- [Biopython](https://biopython.org/)
- [Jupyter](https://jupyter.org/)

---

<p align="center">
  <sub>Built with 🧬 Biopython · 🐍 Python 3.12 · 📓 Jupyter Notebook</sub>
</p>
