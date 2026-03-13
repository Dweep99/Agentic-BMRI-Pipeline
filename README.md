Please note - This repository contains a redacted version of the project. Proprietary prompts and datasets have been removed.
Please contact me if you would like access to the full implementation. 

![BMRI Pipeline Architecture](docs/Agent%20Architecture.png) 

# BMRI Intelligence Pipeline

A multi-agent AI pipeline for detecting business model reinvention from SEC 10-K filings. The system processes multi-year company disclosures, extracts structured business model evidence, classifies reinvention types, computes deterministic transition signals, and generates year-by-year strategic evolution outputs.

This repository is shared as a portfolio project. It demonstrates the architecture, analytics, and engineering design of the system while excluding proprietary prompt packs and the underlying dataset.

## What This Project Does

This system analyzes SEC 10-K filings across multiple years and identifies signals of business model reinvention.

It combines:
- LLM-based structured extraction
- local TF-IDF retrieval
- critic-based validation
- deterministic transition logic
- financial cross-checks

Outputs include:
- structured Business Model Canvas snapshots
- reinvention type classifications
- year-to-year transition detection
- material business model change scores
- AI-generated strategic evolution reports

---

## Architecture

![BMRI Pipeline Architecture](docs/architecture.png)

The pipeline is organized as a sequence of specialized agents and deterministic analytics layers. Retrieval narrows evidence before LLM use, critic agents validate intermediate outputs, and final transition logic is rule-based rather than model-driven.

---

## Why This Problem Matters

Business model reinvention is one of the most important signals in corporate strategy, but identifying it rigorously is difficult. It requires reading long regulatory filings across many years, extracting evidence consistently, and mapping that evidence into a stable analytical framework.

This project automates that process end-to-end: from raw SEC filing text to a structured, evidence-backed timeline of reinvention activity.

---

## System Overview

The pipeline processes company-year filing data through the following stages:

1. **Document parsing**  
   Raw SEC filing text is cleaned and split into paragraph-level objects with stable paragraph IDs.

2. **Retrieval layer**  
   A local TF-IDF index is built for each company-year document and used to retrieve candidate evidence paragraphs.

3. **Selector agent**  
   An LLM filters and prioritizes the most relevant paragraphs from the retrieved candidate pool.

4. **BMCS builder agent**  
   A structured extraction agent creates a 9-field Business Model Canvas snapshot from the filtered evidence.

5. **BMCS critic agent**  
   A critic checks the BMCS output for unsupported claims, invalid citations, and schema violations.

6. **Type rater agent**  
   A classification agent scores each reinvention type using centrality levels.

7. **Type critic agent**  
   A critic downgrades weak or unsupported type assignments.

8. **Deterministic transition engine**  
   Business model type transitions are computed year-over-year using rule-based logic.

9. **Change narration and report generation**  
   Narrative agents describe changes between years and generate a strategic evolution report.

---

## Multi-Model Strategy

Different pipeline steps have different cost, latency, and accuracy requirements. The system uses a multi-model setup rather than a single model for all tasks.

| Component | Role |
|---|---|
| Selector | fast evidence filtering |
| BMCS Builder | structured extraction |
| BMCS Critic | high-accuracy output validation |
| Type Rater | multi-criteria classification |
| Type Critic | fast classification audit |
| Writer | long-form synthesis |
| Editor | revision and consistency checks |

This separation reduces cost where possible while reserving stronger models for high-risk reasoning tasks.

---

## Retrieval Layer

Instead of sending entire 10-K filings into an LLM context window, the system uses a local retrieval layer to narrow the evidence first.

### Retrieval workflow
- Each document is split into paragraphs with stable IDs
- A TF-IDF index is built per document-year
- Multiple retrieval queries are issued across BMCS fields and discovery signals
- Candidate paragraphs are scored, merged, and capped into an evidence pack
- The Selector Agent filters this evidence before downstream generation

This keeps context focused, lowers cost, and reduces hallucination risk.

---

## Schema Validation and Hallucination Control

All structured outputs are validated before they are accepted by the pipeline.

### Validation approach
- Outputs are constrained to structured JSON
- Responses are validated against Pydantic schemas
- Malformed JSON is repaired before validation when possible
- Evidence paragraph IDs are checked against the original evidence pack
- Unsupported claims can be revised, removed, or marked unknown by critic agents

### Key design choice
Presence is not left to free-form model interpretation. The model returns a centrality score, and presence is derived deterministically afterward.

This reduces model discretion in the final classification layer.

---

## Reinvention Framework

The pipeline classifies company-year observations using five business model reinvention categories based on the PwC BMRI framework.

| Type | Strategic Signal | Example |
|---|---|---|
| **XaaS** | Subscription or outcome-based revenue replacing product sales | Microsoft 365, AWS |
| **Digital Products** | Software, data, or AI as a standalone revenue stream | Adobe Creative Cloud |
| **Physical + Connected** | Hardware with embedded software creating ongoing value | Apple Watch |
| **Ecosystem / Platform** | Multi-sided marketplace where third parties transact | Salesforce AppExchange |
| **Channel Innovation** | Structural change in customer acquisition or delivery | DTC, omnichannel |

Each type is scored per year using a centrality scale:

- `0` = absent
- `1` = peripheral
- `2` = core

Presence is then mapped deterministically:

- `2 -> 1`
- `1 -> ?`
- `0 -> 0`

---

## Deterministic Transition Engine

The transition layer is fully rule-based and does not rely on LLM output generation.

For each company, type, and year, the engine computes:

- **Onset** — presence changes into a confirmed active state
- **Continuation** — presence remains active across consecutive years
- **Offset** — presence drops out of an active state
- **Pivot** — one type exits while another becomes active in the same transition window
- **BMRI Pace** — rolling count of active reinvention types over time

This layer is designed to reduce sensitivity to noisy one-year model outputs and anchor key events in deterministic logic.

---

## Material Change Detection

The system also estimates whether the business model changed materially between years.

### Method
- BMCS fields are converted into a canonical text snapshot for each year
- TF-IDF vectors are built within each firm
- Consecutive years are compared using cosine distance
- Distances are ranked within-firm
- Large changes are flagged using a percentile-based threshold

This creates a relative measure of business model change that is comparable within each company over time.

---

## Financial Validation Layer

AI-detected reinvention signals are cross-checked against financial indicators from a separate dataset.

Examples include:
- **Net Asset Turnover (NAT)** as an efficiency proxy
- **PPE Intensity** as a signal of asset-light shifts
- **Intangible Intensity** as a proxy for IP-driven business models
- **Material BMCS Change Score** as a text-based measure of strategic change

This layer is used to support interpretation rather than to override the main classification pipeline.

---

## Engineering Design

### Caching
Every LLM call is cached using a hash of the full request payload. Re-running already-processed company-years avoids repeated API calls.

### Parallelism
The pipeline processes company-years concurrently using thread-based execution with rate-limiting controls.

### Retry handling
LLM calls use configurable retries with exponential backoff and jitter to improve robustness.

### Validation
Structured schemas, citation checks, and critic agents create multiple validation layers before outputs move downstream.

These design choices were made to improve reproducibility, cost control, and reliability.

---

## Output Files

For each company, the pipeline produces structured outputs such as:

| File | Contents |
|---|---|
| `TICKER_bmcs_panel.xlsx` | Business Model Canvas fields across years |
| `TICKER_transitions.xlsx` | Onset, offset, continuation, and pivot events |
| `TICKER_bmcs_change_distances.xlsx` | Year-over-year material change scores |
| `TICKER_type_panel.xlsx` | Reinvention type centrality panel |
| `TICKER_draft_report.md` | Draft strategic evolution report |
| `TICKER_final_report.md` | Revised final report |
| `editor_changelog.json` | Structured record of report edits |

---
<img width="882" height="944" alt="image" src="https://github.com/user-attachments/assets/63120c30-f4d0-4c72-b7e4-0c541a0f0eb2" />

## Technology Stack

`Python 3.11` · `Google Gemini API` · `Pydantic v2` · `scikit-learn` · `pandas` · `numpy` · `jsonrepair` · `tqdm` · `openpyxl` · `python-dotenv`

---

## Setup

```bash
pip install google-genai pydantic pandas numpy scikit-learn tqdm jsonrepair openpyxl python-dotenv
