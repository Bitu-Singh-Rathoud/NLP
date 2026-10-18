# TrialOps-Adaptive AI

**Evidence-grounded clinical trial protocol intelligence and explainable eligibility pre-screening using biomedical NLP, retrieval, and deterministic rules.**

> **Repository status (18 October 2026):** GitHub documentation is being prepared, but the **complete source tree has not yet been uploaded to this repository**. The implementation and experiment results described below are from the locally prepared project documentation; they are **not independently reproducible from this GitHub repository yet**. This notice will be updated after source sync and a fresh checkout test.

> **Academic research demonstration only.** Not for medical advice, diagnosis, actual patient selection, or clinical eligibility decisions. Use **synthetic profiles only** for evaluation.

## What problem does it solve?

Clinical trial inclusion and exclusion criteria are commonly written as free-text paragraphs. Finding an entity such as “HbA1c” is not enough: a system must interpret the measurement, numeric value, comparison operator, unit, negation, and logical scope. This project explores extracting only sufficiently supported criteria into traceable rules and flagging ambiguous text for review.

**Typical flow**

```text
Public trial records (ClinicalTrials.gov API v2)
    -> raw records + timestamps + source URLs
    -> normalized study metadata and eligibility text
    -> deterministic rule parser + biomedical NER
    -> evidence spans and unresolved/unsupported clauses
    -> human review of executable rules
    -> synthetic profile evaluation (true / false / unknown)
    -> explanation with the source condition
```

A separate research-search workflow indexes source passages with **BM25**, **MiniLM embeddings**, and **FAISS**, returning traceable excerpts; optional **FLAN-T5-small** answer drafts are marked experimental and do not override the deterministic eligibility engine.

## Main technology

| Layer | Tools / purpose |
| --- | --- |
| Public research records | ClinicalTrials.gov API v2; optional authorized CTRI import |
| Classical NLP | Unicode normalization, section detection, numeric / unit / negation / Boolean parsing |
| Biomedical named-entity recognition | `d4data/biomedical-ner-all` (pretrained; not fine-tuned here) |
| CHIA entity adaptation experiment | Frozen biomedical encoder + trained linear BIO classification head |
| Semantic retrieval | `sentence-transformers/all-MiniLM-L6-v2`, FAISS |
| Lexical retrieval | BM25; hybrid rank fusion |
| Optional answer draft | `google/flan-t5-small` (not fine-tuned) |
| Backend / frontend | FastAPI, Pydantic / Streamlit |
| Review decisions | SQLite |
| Tests / packaging | pytest / Dockerfiles and Docker Compose |

**Terminology:** The CHIA experiment trained **only a new linear classifier head**, not the complete biomedical encoder. The trained head is an experimental benchmark artifact, not a replacement for the default rule parser. The FLAN-T5 model generates an optional draft; it is **not** a demonstrated schema-constrained LLM eligibility-rule extraction benchmark.

## Documented data and experiments

The locally generated report describes the following observations. These should be reproduced from project artifacts after the repository sync:

- **58 unique** Type 2 Diabetes study records from ClinicalTrials.gov API v2: 32 studies with Indian locations, 30 with United States locations, and 4 with locations in both groups. These are **study-site groupings**, not nationality-based clinical thresholds.
- **0 native CTRI records** in the current published-data snapshot; the available manual importer is not a substitute for verified CTRI provenance.
- **30 synthetic profiles** and two fictional trial protocols retained as regression fixtures.
- **CHIA frozen-encoder head experiment** reported on a held-out entity test set:
  - Training-derived dictionary: exact-span F1 **0.4253**
  - Original pretrained biomedical NER: exact-span F1 **0.1265**
  - Frozen encoder + newly trained head: exact-span F1 **0.3212**
- **Complete text-rule parsing:** **18 of 983** assessed real criteria (approximately **1.83%**). The remaining text is not silently treated as a complete rule; broad clinical eligibility inference is **not established**.
- The local report states **150 passing software tests** and **17 UI action smoke checks**. These numbers reflect reported software testing, **not clinical accuracy**.

These results are **not** a controlled three-way structured-rule comparison. The current artifacts do **not** establish a large human-verified rule-level gold set, fully fine-tuned biomedical encoder, calibrated selective abstention, or proven country/phase domain adaptation.

## Meaning of eligibility outputs

- `MATCHES_KNOWN_CRITERIA`: the supplied, approved, supported rules evaluated as matching; this does **not** establish complete trial eligibility.
- `DOES_NOT_MATCH_KNOWN_CRITERIA`: at least one known executable condition failed.
- `INSUFFICIENT_INFORMATION`: missing, unresolved, unsupported, or unapproved conditions prevent a definitive evaluation of known constraints.

Missing clinical information is never automatically considered false. Explicit source evidence and review are required before rule execution.

## Local setup (once project source is synced)

The existing Windows development environment is documented under:

`C:\Users\LENOVO\OneDrive\Documents\ChatGPT\NLP`

From Windows PowerShell in the project root:

```powershell
.\scripts\start.ps1
.\scripts\smoke.ps1 -Full
.\.venv\Scripts\python.exe -m scripts.verify --live
.\scripts\stop.ps1
```

Expected local services when the project is installed and running:

- Streamlit: http://127.0.0.1:8501
- FastAPI documentation: http://127.0.0.1:8000/docs

**These commands require the source files, local dependencies, and any model weights/data needed by the selected feature; they cannot run from the current documentation-only GitHub state.**

### Docker

```powershell
docker compose config --quiet
docker compose up --build
```

The local documentation reports that Compose configuration validation passed, but a Docker Desktop Linux engine problem prevented an actual image build. **Container execution is not yet verified.**

## Expected source layout after synchronization

```text
backend/app/            FastAPI, NLP, model inference, rules, retrieval, review
frontend/               Streamlit pages
data/                   Source-derived data / synthetic fixtures (subject to licenses)
experiments/            CHIA adaptation and evaluation scripts
reports/                Recorded metrics and test artifacts
scripts/                Windows startup, smoke tests, verification
docs/                   Methodology, project explanation, limitations
tests/                  pytest suite
Dockerfile(s)           Container packaging
docker-compose.yml      Backend/frontend services
```

**Large or restricted materials are intentionally excluded from Git:** virtual environments, cached model weights, downloaded corpora requiring attribution or access, keys/tokens, local review databases, and large generated artifacts. A reproducible installation process should fetch eligible public resources and record source/license information.

## Scientific limitations

1. Complete-criterion rule extraction coverage is currently low.
2. Entity F1 is **not** patient-matching accuracy or complete-rule accuracy.
3. The documented transformer adaptation is a frozen-encoder **head training** experiment.
4. The project has not completed a three-way extraction comparison on independently human-verified structured-rule labels.
5. Regional site filters cannot change trial eligibility thresholds. Trials with Indian and US sites may share the same global protocol.
6. Synthetic profiles are not evidence of safety, clinical accuracy, or real patient benefit.
7. Docker runtime verification remains pending.

## Data sources and model credits

- [ClinicalTrials.gov](https://clinicaltrials.gov/) and [API documentation](https://clinicaltrials.gov/data-api/api)
- [CHIA dataset](https://huggingface.co/datasets/bigbio/chia) — check original annotations, attribution, and applicable licensing before redistribution
- [Biomedical NER model](https://huggingface.co/d4data/biomedical-ner-all)
- [MiniLM sentence embeddings](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2)
- [FLAN-T5-small](https://huggingface.co/google/flan-t5-small)
- [scispaCy](https://github.com/allenai/scispacy)

## Reproducibility and authorship

Development involved AI assistance. Repository code, recorded experiments, evaluation data lineage, and independent reruns—not documentation alone—are the evidence for technical claims.

**Project status:** source upload and a clean-clone verification are the immediate next steps. Until then, treat this repository as documentation rather than a runnable release.
