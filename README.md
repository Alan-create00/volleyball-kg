# Volleyball Knowledge Graph — Web Mining & Semantics Project

**Domain:** International Men's Volleyball (FIVB)  
**Sources:** Wikipedia API + Wikidata SPARQL  
**Stack:** Python 3.10 · rdflib · PyKEEN · OWLReady2 · Anthropic API · Google Colab

---

## Project Structure

```
volleyball-kg/
├── notebooks/
│   ├── lab1_crawl_ner.ipynb        # Crawler + NER
│   ├── lab2_kb_construction.ipynb  # KB Construction & Alignment
│   ├── lab3_reasoning_kge.ipynb    # SWRL Reasoning + KGE
│   └── lab4_rag.ipynb              # RAG over RDF/SPARQL
├── kg_artifacts/
│   ├── ontology.ttl                # OWL ontology (8 classes, 12 properties)
│   ├── initial_kb.nt               # Initial RDF graph (5,571 triples)
│   ├── expanded_clean.nt           # Expanded KB (84,016 triples)
│   ├── alignment.ttl               # Predicate alignment + owl:sameAs
│   ├── alignment_table.csv         # Entity linking results
│   └── kb_statistics.txt           # KB volume stats
├── data/
│   ├── train.txt                   # KGE training split (67,212 triples)
│   ├── valid.txt                   # KGE validation split (6,799 triples)
│   ├── test.txt                    # KGE test split (6,868 triples)
│   └── samples/                    # Sample data files
├── rag/
│   ├── evaluation_table.csv        # 20-question RAG evaluation
│   └── discussion.txt              # RAG analysis report
├── reports/
│   └── final_report.pdf            # Final project report
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Installation

```bash
pip install rdflib SPARQLWrapper pandas scikit-learn matplotlib
pip install pykeen torch
pip install owlready2
pip install anthropic
```

Or use the provided requirements file:

```bash
pip install -r requirements.txt
```

**Hardware requirements:**
- Labs 1–2: any machine (CPU only, ~2 GB RAM)
- Lab 3 (KGE training): GPU recommended — T4 on Google Colab works well
- Lab 4 (RAG): CPU only, requires Anthropic API key

---

## How to Run Each Module

### Lab 1 — Crawl + NER
Open `notebooks/lab1_crawl_ner.ipynb` in Google Colab.  
Requires: Google Drive mounted at `/content/drive/MyDrive/volleyball-kg/`  
Output: `data/crawler_output.jsonl`, `data/extracted_knowledge.csv`

### Lab 2 — KB Construction
Open `notebooks/lab2_kb_construction.ipynb` in Google Colab.  
Requires: Lab 1 outputs in Drive.  
Output: `kg_artifacts/expanded_clean.nt` (84k triples, 51 predicates)

### Lab 3 — Reasoning + KGE
Open `notebooks/lab3_reasoning_kge.ipynb` in Google Colab.  
**Set runtime to GPU** before running: Runtime → Change runtime type → T4 GPU  
Requires: `expanded_clean.nt` + `family.owl` in Drive (`volleyball-kg/family.owl`)  
Output: `data/train.txt`, `data/valid.txt`, `data/test.txt` + trained models

### Lab 4 — RAG Demo
Open `notebooks/lab4_rag.ipynb` in Google Colab.  
Requires: Anthropic API key in Colab Secrets as `ANTHROPIC_API_KEY`  
Output: `rag/evaluation_table.csv`, `rag/evaluation_chart.png`

**To run the interactive chatbot**, uncomment the `while True:` block at the end of the demo cell.

---

## RAG Demo

The RAG system takes a natural language question, generates SPARQL via Claude Haiku,
executes it on the local knowledge graph with rdflib, and self-repairs if needed.

**Example:**
```
Question: Which players represent France in volleyball?

With KB (SPARQL + rdflib):
  - Earvin Ngapeth
  - Jenia Grebennikov
  - Benjamin Toniutti
  - Kévin Le Roux
  ... (20 results total)
```

**RAG evaluation: 17/20 (85%) on 20 test questions.**

---

## Key Results

| Lab | Key Output |
|-----|-----------|
| Lab 1 | 2,826 entities, 2,710 relations from 35 Wikipedia pages |
| Lab 2 | 84,016 triples, 51 predicates, 135 owl:sameAs links |
| Lab 3 | RotatE MRR=0.185 (best), TransE MRR=0.024, ComplEx MRR=0.099 |
| Lab 4 | 17/20 (85%) RAG success, 5 self-repairs triggered |

---

## Notes

- The Wikidata SPARQL endpoint is queried during Labs 1, 2, and 4.
  A `User-Agent` header is always included as required by Wikimedia policy.
- `family.owl` must be placed manually in Google Drive at
  `MyDrive/volleyball-kg/family.owl` before running Lab 3.
- KGE training time: ~15 minutes per model on T4 GPU (100 epochs, 67k triples).
