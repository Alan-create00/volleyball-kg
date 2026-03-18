# 🏐 Volleyball Knowledge Graph

A full Knowledge Graph pipeline applied to **international men's volleyball (FIVB)**.  
Built as part of the *Web Mining & Semantics* course project.

---

## 📋 Project Overview

This project builds a complete Knowledge Graph pipeline in 4 stages:

| Stage | Description | Notebook |
|-------|-------------|----------|
| Lab 1 | Web Crawling + NER | `notebooks/lab1_crawl_ner.ipynb` |
| Lab 2 | KB Construction + Alignment + Expansion | `notebooks/lab2_kb_construction.ipynb` |
| Lab 3 | SWRL Reasoning + KGE | `notebooks/lab3_reasoning_kge.ipynb` |
| Lab 4 | RAG over RDF/SPARQL | `notebooks/lab4_rag.ipynb` |

---

## 📁 Repository Structure

```
volleyball-kg/
├── src/
│   ├── crawl/          # Wikipedia crawler + Wikidata fetcher
│   ├── ie/             # NER extraction pipeline
│   ├── kg/             # RDF graph + ontology + alignment
│   ├── reason/         # SWRL rules (OWLReady2)
│   ├── kge/            # KGE training + evaluation (PyKEEN)
│   └── rag/            # RAG pipeline (NL→SPARQL + self-repair)
├── data/
│   ├── samples/        # Sample data for reproducibility
│   └── README.md
├── kg_artifacts/
│   ├── ontology.ttl    # OWL ontology
│   ├── expanded.nt     # Expanded KB (N-Triples)
│   └── alignment.ttl   # Alignment file
├── reports/
│   └── final_report.pdf
├── notebooks/          # Google Colab notebooks (one per lab)
├── README.md
├── requirements.txt
├── .gitignore
└── LICENSE
```

---

## ⚙️ Installation

### Option A — Google Colab (recommended)
Each notebook contains a setup cell that installs all dependencies automatically.

### Option B — Local
```bash
git clone https://github.com/YOUR_USERNAME/volleyball-kg.git
cd volleyball-kg
pip install -r requirements.txt
python -m spacy download en_core_web_trf
```

> ⚠️ Java is required for OWLReady2 (SWRL reasoning). Install [JDK 11+](https://adoptium.net/).

---

## 🚀 How to Run Each Module

### Lab 1 — Crawl + NER
Open `notebooks/lab1_crawl_ner.ipynb` in Colab.  
Outputs: `data/crawler_output.jsonl`, `data/extracted_knowledge.csv`

### Lab 2 — KB Construction
Open `notebooks/lab2_kb_construction.ipynb` in Colab.  
Outputs: `kg_artifacts/ontology.ttl`, `kg_artifacts/expanded.nt`, `kg_artifacts/alignment.ttl`

### Lab 3 — Reasoning + KGE
Open `notebooks/lab3_reasoning_kge.ipynb` in Colab.  
Outputs: `data/train.txt`, `data/valid.txt`, `data/test.txt`, evaluation metrics

### Lab 4 — RAG Demo
Open `notebooks/lab4_rag.ipynb` in Colab.  
Run the CLI demo cell at the bottom of the notebook.

---

## 🖥️ Hardware Requirements

| Task | Minimum | Recommended |
|------|---------|-------------|
| Crawling + NER | CPU | CPU |
| KB Construction | CPU | CPU |
| KGE Training | CPU (slow) | **GPU (Colab T4)** |
| RAG Pipeline | CPU | CPU |

---

## 📸 Screenshots

*(To be added after Lab 4)*

---

## 📊 KB Statistics

*(To be filled after Lab 2)*

---

## 👥 Authors

- *[Your Name]*  
  Course: Web Mining & Semantics — ESILV
