# Volleyball Knowledge Graph
### Web Mining & Semantics — ESILV

A knowledge graph built around international men's volleyball (FIVB), covering the full pipeline
from Wikipedia crawling to a RAG-powered chatbot that answers questions in natural language.

---

## Pipeline Overview

This project follows a 4-step pipeline where each lab produces the files the next one needs.
Run them in order.

```
Wikipedia API (43 pages attempted)
       |
       v
  Lab 1 -- Crawl + NER
  35 pages crawled · 2,825 entities · 2,510 relations
       |
       v
  Lab 2 -- KB Construction & Alignment
  5,559 initial triples → 81,836 expanded · 49 predicates · 129 owl:sameAs links
       |
       v
  Lab 3 -- SWRL Reasoning & KGE
  2 SWRL rules · RotatE MRR=0.1714 (best) · 65,561 train / 6,578 valid / 6,607 test
       |
       v
  Lab 4 -- RAG over RDF
  16/20 (80%) accuracy · 6 self-repairs triggered · claude-haiku-4-5-20251001
```

**Stack:** Python 3.10 · rdflib · PyKEEN · OWLReady2 · Anthropic API · Google Colab

---

## Repository Structure

```
volleyball-kg/
├── notebooks/
│   ├── lab1_crawl_ner.ipynb           # Crawler + NER
│   ├── lab2_kb_construction.ipynb     # KB construction, alignment, expansion
│   ├── lab3_reasoning_kge.ipynb       # SWRL reasoning + KGE training
│   └── lab4_rag.ipynb                 # RAG: NL -> SPARQL + chatbot
│
├── kg_artifacts/
│   ├── ontology.ttl                   # OWL ontology (8 classes, 12 object properties)
│   ├── initial_kb.nt                  # Initial RDF graph (5,559 triples)
│   ├── expanded_clean.nt              # Cleaned expanded KB (81,836 triples, 49 predicates)
│   ├── alignment.ttl                  # 8 equivalent properties + 129 owl:sameAs links
│   ├── alignment_table.csv            # Entity linking results (200 entities processed)
│   └── kb_statistics.txt             # KB volume statistics
│
├── data/
│   ├── train.txt                      # KGE training split (65,561 triples)
│   ├── valid.txt                      # KGE validation split (6,578 triples)
│   ├── test.txt                       # KGE test split (6,607 triples)
│   └── samples/                       # Sample files for quick testing
│
├── rag/
│   ├── evaluation_table.csv           # RAG evaluation on 20 questions
│   └── discussion.txt                 # Results analysis
│
├── reports/
│   └── final_report.pdf               # Final report (6-10 pages)
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
- Lab 3 (KGE training): GPU strongly recommended -- Colab T4 works well (~15 min per model)
- Lab 4 (RAG): CPU only, requires an Anthropic API key

---

## Running the Notebooks

Important: run the labs in order. Each notebook generates the files the next one depends on.

Before starting, mount Google Drive and make sure this folder exists:

```
MyDrive/volleyball-kg/
```

Also place `family.owl` in that folder manually -- it is required for Lab 3 and is not
generated automatically. The file uses OWL 1.0 typed individual syntax; the notebook
applies two fixes automatically (removes the remote owl:imports line, uses class.instances()
instead of individuals()).

---

### Lab 1 -- Crawl + NER

**Notebook:** `notebooks/lab1_crawl_ner.ipynb`

Crawls Wikipedia pages about FIVB men's volleyball (43 pages attempted, 35 successfully
retrieved), queries Wikidata SPARQL for structured relations, and runs NER with spaCy
(en_core_web_trf) to extract named entities.

**Key outputs:**
```
Pages crawled (Wikipedia):  35
Wikidata entity records:  1,300
Wikidata relation triples: 2,300
Dep parsing verb relations:  220
Unique entities (cleaned):  2,825
Total unique relations:    2,510
```

Entity breakdown: ORG (816) · PERSON (555) · DATE (535) · EVENT (534) · GPE (373) · LOC (12)

Relation sources: wikidata_structured (2,300 · 91.6%) · verb+prep (154) · subj-verb-obj (56)

**Files produced:**
- `data/crawler_output.jsonl`
- `data/wikidata_output.jsonl`
- `data/extracted_knowledge.csv`
- `data/extracted_relations.csv`

---

### Lab 2 -- KB Construction

**Notebook:** `notebooks/lab2_kb_construction.ipynb`

Builds the initial RDF graph from Lab 1 data, aligns entities with Wikidata (200 entities
processed, 129 linked), aligns predicates manually (8 owl:equivalentProperty), then expands
the KB via three SPARQL strategies (1-hop, predicate-controlled, 2-hop).

**Key outputs:**
```
Initial KB:      5,559 triples · 2,825 entities
Alignment:         129 owl:sameAs links · 8 equivalent properties
Expanded KB:    81,836 triples · 36,088 entities · 49 predicates
File size:          10.0 MB
```

Size validation: triples OK (target 50k-200k) · entities OK (target 5k-30k) · predicates OK (target 50+)

**Files produced:**
- `kg_artifacts/ontology.ttl` (8 classes, 12 object properties, 3 datatype properties)
- `kg_artifacts/initial_kb.nt`
- `kg_artifacts/alignment.ttl`
- `kg_artifacts/alignment_table.csv`
- `kg_artifacts/expanded_clean.nt`
- `kg_artifacts/kb_statistics.txt`

---

### Lab 3 -- SWRL Reasoning & KGE

**Notebook:** `notebooks/lab3_reasoning_kge.ipynb`

Switch the Colab runtime to GPU before running: Runtime -> Change runtime type -> T4 GPU

**Part 1 -- SWRL Reasoning** (OWLReady2 + Pellet/HermiT):

Two rules are applied:

```
Rule 1 (family.owl):
  Person(?p) ∧ age(?p, ?a) ∧ swrlb:greaterThan(?a, 60) → oldPerson(?p)
  Result: 2 instances inferred (Peter age=70, Marie age=69)

Rule 2 (Volleyball KB):
  Player(?p) ∧ representsCountry(?p, ?c) → playsForNationalTeamOf(?p, ?c)
  Result: 10 triples inferred (Ngapeth→France, Kurek→Poland, Giba→Brazil, ...)
```

**Part 2 -- KGE (PyKEEN, Tesla T4 GPU):**

```
Dataset: 81,952 triples · 36,785 entities · 49 relations
Split:   65,561 train (80%) · 6,578 valid (8%) · 6,607 test (8%)
Config:  dim=100 · lr=0.01 · batch=512 · epochs=100
```

**Requires:** `family.owl` at `MyDrive/volleyball-kg/family.owl`

**Files produced:**
- `data/train.txt`, `data/valid.txt`, `data/test.txt`
- Trained model checkpoints (PyKEEN)
- `kge/model_comparison.png`, `kge/tsne_embeddings.png`

---

### Lab 4 -- RAG over RDF

**Notebook:** `notebooks/lab4_rag.ipynb`

The RAG system takes a natural language question, generates a SPARQL query via Claude Haiku,
executes it on the local graph with rdflib, and self-repairs up to 2 times if the query
fails or returns 0 results.

**Requires:** `ANTHROPIC_API_KEY` in Colab Secrets (key icon in the left panel)

**Files produced:**
- `rag/evaluation_table.csv`
- `rag/evaluation_chart.png`
- `rag/discussion.txt`

To launch the interactive chatbot, uncomment the `while True:` block at the end of the demo cell.

---

## RAG Demo

```
Question: Which players represent France in volleyball?

Without KB (Claude Haiku only):
  France has several notable volleyball players, including Earvin Ngapeth,
  Olivier Nyako, and Yacine Louati... (unverifiable, may be outdated)

With KB (SPARQL + rdflib):
  - Youssef Krou          - Jenia Grebennikov
  - Nicolas Le Goff       - Stephan Boyer
  - Kevin Le Roux         - Earvin Ngapeth
  - Benjamin Toniutti     - Axel Bonami
  ... (20 grounded results from the KB)
```

---

## Key Results

| Lab | Output |
|-----|--------|
| Lab 1 -- Crawl + NER | 35/43 pages · 2,825 entities · 2,510 relations |
| Lab 2 -- KB Construction | 81,836 triples · 49 predicates · 129 owl:sameAs links |
| Lab 3 -- KGE | RotatE MRR=0.1714 (best) · ComplEx MRR=0.0822 · TransE MRR=0.0463 |
| Lab 4 -- RAG | 16/20 (80%) accuracy · 6 self-repairs (2 successful) |

---

## Embedding Models

| Model | Type | MRR | Hits@1 | Hits@3 | Hits@10 |
|-------|------|-----|--------|--------|---------|
| TransE | Translational | 0.0463 | 0.0160 | 0.0502 | 0.1006 |
| ComplEx | Complex bilinear | 0.0822 | 0.0608 | 0.0864 | 0.1184 |
| RotatE | Rotation in C | **0.1714** | **0.1349** | **0.1803** | **0.2388** |

RotatE achieves the best results across all metrics. Its ability to model symmetric,
antisymmetric, and compositional relations makes it well-suited to a sports knowledge
graph where player-country, team-federation, and tournament-edition chains are common.

Embedding arithmetic confirms partial alignment with the SWRL rule:
cosine_similarity(representsCountry, playsFor) = 0.676, versus 0.511 for an unrelated pair.

---

## Notes

- Wikidata SPARQL endpoints are queried during Labs 1, 2, and 4.
  A User-Agent header is always included as required by Wikimedia policy.
- `family.owl` must be placed manually in Google Drive before running Lab 3.
  The notebook patches the file automatically (removes remote owl:imports, fixes OWL 1.0 syntax).
- KGE training time: ~15 minutes per model on T4 GPU (100 epochs, 65k triples).
- If Wikidata returns 429 errors, add time.sleep() calls between requests.
- All notebook paths assume Drive is mounted at `/content/drive/MyDrive/volleyball-kg/`.
  Adjust if your folder name differs.
- The RAG model used is `claude-haiku-4-5-20251001`. Make sure your Anthropic API key
  has access to this model before running Lab 4.
