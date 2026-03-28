# Volleyball Knowledge Graph
### Web Mining & Semantics — ESILV

A knowledge graph built around international men's volleyball (FIVB), covering the full pipeline
from Wikipedia crawling to a RAG-powered chatbot that answers questions in natural language.

---

## Pipeline Overview

This project follows a 4-step pipeline where each lab produces the files the next one needs.
Run them in order.

```
Wikipedia / Wikidata
       |
       v
  Lab 1 -- Crawl + NER
  (entity extraction, cleaning)
       |
       v
  Lab 2 -- KB Construction & Alignment
  (RDF, ontology, Wikidata alignment, SPARQL expansion)
       |
       v
  Lab 3 -- Reasoning & KGE
  (SWRL rules, TransE / RotatE / ComplEx embeddings)
       |
       v
  Lab 4 -- RAG over RDF
  (NL -> SPARQL via Claude, self-repair, chatbot)
```

**Stack:** Python 3.10 · rdflib · PyKEEN · OWLReady2 · Anthropic API · Google Colab

---

## Repository Structure

```
volleyball-kg/
├── notebooks/
│   ├── lab1_crawl_ner.ipynb          # Crawler + NER
│   ├── lab2_kb_construction.ipynb    # KB construction, alignment, expansion
│   ├── lab3_reasoning_kge.ipynb      # SWRL reasoning + KGE training
│   └── lab4_rag.ipynb                # RAG: NL -> SPARQL + chatbot
│
├── kg_artifacts/
│   ├── ontology.ttl                  # OWL ontology (8 classes, 12 properties)
│   ├── initial_kb.nt                 # Initial RDF graph (5,571 triples)
│   ├── expanded_clean.nt             # Cleaned expanded KB (84,016 triples)
│   ├── alignment.ttl                 # Predicate alignment + owl:sameAs links
│   ├── alignment_table.csv           # Entity linking results
│   └── kb_statistics.txt            # KB volume statistics
│
├── data/
│   ├── train.txt                     # KGE training split (67,212 triples)
│   ├── valid.txt                     # KGE validation split (6,799 triples)
│   ├── test.txt                      # KGE test split (6,868 triples)
│   └── samples/                      # Sample files for quick testing
│
├── rag/
│   ├── evaluation_table.csv          # RAG evaluation on 20 questions
│   └── discussion.txt                # Results analysis
│
├── reports/
│   └── final_report.pdf              # Final report (6-10 pages)
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Installation

```bash
# Core dependencies
pip install rdflib SPARQLWrapper pandas scikit-learn matplotlib

# KGE (GPU recommended for Lab 3)
pip install pykeen torch

# OWL reasoning
pip install owlready2

# Claude API (for RAG in Lab 4)
pip install anthropic
```

Or install everything at once:

```bash
pip install -r requirements.txt
```

**Hardware requirements:**
- Labs 1 & 2: CPU only, ~2 GB RAM
- Lab 3 (KGE training): GPU strongly recommended -- Colab T4 works well
- Lab 4 (RAG): CPU only, requires an Anthropic API key

---

## Running the Notebooks

Important: run the labs in order. Each notebook generates the files the next one depends on.

Before starting, mount Google Drive and make sure this folder exists:

```
MyDrive/volleyball-kg/
```

Also place `family.owl` in that folder manually -- it is required for Lab 3 and is not generated automatically.

---

### Lab 1 -- Crawl + NER

**Notebook:** `notebooks/lab1_crawl_ner.ipynb`

Crawls Wikipedia pages about volleyball, extracts named entities (players, teams, tournaments),
and cleans the raw data.

**Outputs:**
- `data/crawler_output.jsonl`
- `data/extracted_knowledge.csv`

---

### Lab 2 -- KB Construction

**Notebook:** `notebooks/lab2_kb_construction.ipynb`

Builds the RDF graph, aligns it with Wikidata via SPARQL, and expands it to ~84,000 triples.

**Outputs:**
- `kg_artifacts/ontology.ttl`
- `kg_artifacts/expanded_clean.nt` (~84k triples, 51 predicates)
- `kg_artifacts/alignment.ttl` + `alignment_table.csv`

---

### Lab 3 -- Reasoning & KGE

**Notebook:** `notebooks/lab3_reasoning_kge.ipynb`

Switch the Colab runtime to GPU before running: Runtime -> Change runtime type -> T4 GPU

Applies SWRL rules on `family.owl`, then trains Knowledge Graph Embedding models
(TransE, RotatE, ComplEx) for link prediction.

**Requires:** `family.owl` at `MyDrive/volleyball-kg/family.owl`

**Outputs:**
- `data/train.txt`, `data/valid.txt`, `data/test.txt`
- Trained model checkpoints (PyKEEN)

---

### Lab 4 -- RAG over RDF

**Notebook:** `notebooks/lab4_rag.ipynb`

The RAG system takes a natural language question, generates a SPARQL query via Claude,
executes it on the local graph with rdflib, and self-repairs if the query fails.

**Requires:** `ANTHROPIC_API_KEY` in Colab Secrets (key icon in the left panel)

**Outputs:**
- `rag/evaluation_table.csv`
- `rag/evaluation_chart.png`

To launch the interactive chatbot, uncomment the `while True:` block at the end of the demo cell.

---

## RAG Demo

```
Question: Which players represent France in volleyball?

Answer (via SPARQL + rdflib):
  - Earvin Ngapeth
  - Jenia Grebennikov
  - Benjamin Toniutti
  - Kevin Le Roux
  ... (20 results total)
```

RAG evaluation: 17/20 (85%) on 20 test questions.

---

## Key Results

| Lab | Output |
|-----|--------|
| Lab 1 -- Crawl + NER | 2,826 entities, 2,710 relations from 35 Wikipedia pages |
| Lab 2 -- KB Construction | 84,016 triples, 51 predicates, 135 owl:sameAs links |
| Lab 3 -- KGE | RotatE MRR=0.185 (best), TransE MRR=0.024, ComplEx MRR=0.099 |
| Lab 4 -- RAG | 85% accuracy, 5 self-repairs triggered |

---

## Embedding Models

| Model | Type | MRR |
|-------|------|-----|
| TransE | Translational | 0.024 |
| ComplEx | Complex bilinear | 0.099 |
| RotatE | Rotation in C | 0.185 |

RotatE achieves the best results thanks to its ability to model symmetric, antisymmetric,
and compositional relations -- all of which are common in a sports knowledge graph
(player -> team -> competition).

---

## Notes

- Wikidata SPARQL endpoints are queried during Labs 1, 2, and 4.
  A User-Agent header is always included as required by Wikimedia policy.
- `family.owl` must be placed manually in Google Drive before running Lab 3.
- KGE training time: ~15 minutes per model on T4 GPU (100 epochs, 67k triples).
- If Wikidata returns 429 errors, add time.sleep() calls between requests.
- All notebook paths assume Drive is mounted at `/content/drive/MyDrive/volleyball-kg/`.
  Adjust if your folder name differs.
