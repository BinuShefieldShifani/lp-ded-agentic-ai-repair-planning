# 🔧 Knowledge-Based Agentic AI Decision Support System for LP-DED Repair Planning

> On-premise agentic AI system for **Laser Powder Directed Energy Deposition (LP-DED) repair planning** of aerospace components — developed as an MS thesis in collaboration with **GKN Aerospace**.

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![LangGraph](https://img.shields.io/badge/LangGraph-Agentic_Pipeline-orange)
![ChromaDB](https://img.shields.io/badge/ChromaDB-Knowledge_Base-green)
![RAG](https://img.shields.io/badge/RAG-FAISS_Retrieval-purple)
![Domain](https://img.shields.io/badge/Domain-Aerospace_AM-red)
![Confidential](https://img.shields.io/badge/Code-Confidential_(GKN_IP)-lightgrey)

> ⚠️ **Note:** This repository contains no code. The system was developed under contract with GKN Aerospace and the implementation is subject to IP restrictions. This README documents the architecture, methodology, and outcomes for portfolio purposes.

---

## 📌 Overview

Laser Powder Directed Energy Deposition (LP-DED) is an advanced additive manufacturing technique used to repair high-value aerospace components — turbine blades, fan blades, and compressor cases — made from **Ti-6Al-4V** (titanium alloy) and **Inconel 718** (nickel superalloy).

Despite being a proven technology, selecting optimal repair parameters is entirely operator-dependent, relying on repeated physical trials that are:
- **Expensive** — each trial consumes costly aerospace-grade materials and machine time
- **Non-systematic** — knowledge is lost when engineers change roles or face new damage types
- **Non-transferable** — no mechanism exists to capture and validate process knowledge for future use

This thesis addresses that gap by developing an **on-premise, knowledge-driven, agentic AI decision-support system** for LP-DED pre-deposition repair planning — providing traceable, knowledge-grounded parameter recommendations without cloud infrastructure or proprietary databases.

---

## 🎯 What the System Does

Given a damaged aerospace component and its material type, the system:

1. Retrieves relevant process parameters from a structured knowledge base built from peer-reviewed LP-DED literature
2. Verifies consistency of retrieved information across three knowledge layers
3. Generates grounded parameter suggestions using a critic-verified generation process
4. Classifies engineer responses into intent categories and iterates accordingly
5. Expands the knowledge base with engineer-approved instances through a two-phase approval process

The result is a **traceable starting point for repair planning** — each recommendation is directly grounded in published research, not black-box model weights.

---

## 🏗️ System Architecture

The system consists of two major pipelines:

### Pipeline 1 — Knowledge Base Construction

```
Peer-reviewed LP-DED literature (PDFs)
              │
              ▼
    ┌─────────────────────────┐
    │   Document Ingestion    │
    │   & Preprocessing       │
    └──────────┬──────────────┘
               │
               ▼
    ┌─────────────────────────┐
    │   Local LLM Extraction  │
    │                         │
    │  Extracts structured    │
    │  process parameters     │
    │  from article text      │
    └──────────┬──────────────┘
               │
               ▼
    ┌──────────────────────────────────────┐
    │         ChromaDB Knowledge Base      │
    │         (380 entries total)          │
    │                                      │
    │  Layer 1 — Single-bead parameters    │
    │  Layer 2 — Multi-layer deposition    │
    │  Layer 3 — Repair case studies       │
    └──────────────────────────────────────┘
```

### Pipeline 2 — Runtime Agentic System (LangGraph)

```
Engineer Input
(material + damage description)
              │
              ▼
┌─────────────────────────────────────────────┐
│              LangGraph Agent Pipeline        │
│                                             │
│  ┌──────────────┐    ┌──────────────────┐   │
│  │  Retrieval   │    │   Consistency    │   │
│  │  Agent       │───►│   Verification   │   │
│  │              │    │   Agent          │   │
│  │ FAISS-based  │    │                  │   │
│  │ dual-query   │    │ 3-condition LLM  │   │
│  │ across all   │    │ grounding check  │   │
│  │ 3 layers     │    └────────┬─────────┘   │
│  └──────────────┘             │             │
│                               ▼             │
│                    ┌──────────────────┐     │
│                    │  Suggestion      │     │
│                    │  Generator +     │     │
│                    │  Critic Agent    │     │
│                    │                  │     │
│                    │  Generates and   │     │
│                    │  self-verifies   │     │
│                    │  parameter       │     │
│                    │  recommendations │     │
│                    └────────┬─────────┘     │
│                             │               │
│                             ▼               │
│                    ┌──────────────────┐     │
│                    │  Intent          │     │
│                    │  Classifier      │     │
│                    │  Agent           │     │
│                    │                  │     │
│                    │  Classifies      │     │
│                    │  engineer        │     │
│                    │  response into   │     │
│                    │  3 categories    │     │
│                    └────────┬─────────┘     │
│                             │               │
└─────────────────────────────┼───────────────┘
                              │
                              ▼
              ┌───────────────────────────┐
              │  Knowledge Base Expansion │
              │                           │
              │  Two-phase approval:      │
              │  1. Engineer review       │
              │  2. Write-back to         │
              │     ChromaDB on approval  │
              └───────────────────────────┘
```

---

## 🧠 Key Technical Contributions

### 1. Three-Layer Knowledge Architecture
The ChromaDB knowledge base separates LP-DED knowledge into three distinct layers — single-bead parameters, multi-layer deposition, and repair case studies — enabling targeted retrieval at the appropriate level of specificity for each query.

### 2. Dual-Query Layer 1 Retrieval
A novel dual-query approach to Layer 1 retrieval that addresses a known limitation of standard RAG — single queries frequently miss relevant entries when parameter terminology varies across publications. Dual querying with result merging and deduplication significantly improves recall.

### 3. Three-Condition LLM Grounding
Retrieved parameters are validated against three grounding conditions before being passed to the suggestion generator — ensuring that recommendations are not hallucinated or extrapolated beyond what the knowledge base actually supports.

### 4. Critic-Verified Suggestion Generation
A generator-critic pattern where one agent produces parameter recommendations and a second agent independently evaluates them against retrieved evidence — rejecting or requesting revision before presenting to the engineer.

### 5. Round-Based Knowledge Base Expansion
Engineer-approved parameter sets are written back into ChromaDB through a two-phase approval process — making the system progressively more knowledgeable through validated operational experience, not just static literature.

---

## 📊 System Specifications

| Property | Details |
|---|---|
| **Target materials** | Ti-6Al-4V, Inconel 718 |
| **Process scope** | Pre-deposition LP-DED repair planning |
| **Knowledge base size** | 380 entries across 3 layers |
| **Knowledge source** | Peer-reviewed LP-DED literature |
| **Deployment** | Fully on-premise — no cloud infrastructure |
| **LLM** | Local (on-premise, not disclosed) |
| **Vector database** | ChromaDB |
| **Retrieval** | FAISS-based similarity search |
| **Agent framework** | LangGraph |
| **Infrastructure dependency** | None — fully air-gapped capable |

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Agent orchestration | LangGraph |
| Vector database | ChromaDB |
| Retrieval | FAISS (Facebook AI Similarity Search) |
| LLM | Local on-premise model |
| Knowledge extraction | LLM-based structured extraction from PDFs |
| Language | Python |

---

## ✈️ Domain Context — Why This Matters

LP-DED repair of aerospace components is a safety-critical process. A repaired turbine blade that fails in service can cause catastrophic engine damage. The industry therefore requires:

- **Traceability** — every parameter decision must be justifiable
- **Consistency** — same damage type should produce consistent repair approaches
- **Knowledge retention** — institutional knowledge must survive personnel changes

Existing solutions are either fully manual (engineer expertise) or black-box ML models that cannot explain their recommendations. This system occupies a unique position — **AI-assisted but fully traceable**, grounded in published research rather than opaque model weights.

---

## 📝 Thesis Information

| Property | Details |
|---|---|
| **Title** | Knowledge-Based Agentic AI Decision Support System for LP-DED Repair and Process Design |
| **Degree** | MS AI & Automation |
| **University** | University West, Trollhättan, Sweden |
| **Industry Partner** | GKN Aerospace |
| **Year** | 2025 |
| **Availability** | Confidential — not publicly published |

---

## 👤 Author

**Binu Shefield Shifani**
Software Engineer (5 years, Cognizant Technology Solutions)
MS AI & Automation · University West, Trollhättan, Sweden

[![GitHub](https://img.shields.io/badge/GitHub-BinuShefieldShifani-black?logo=github)](https://github.com/BinuShefieldShifani)

---

## 📄 Note on Code Availability

The implementation code for this project was developed under contract with GKN Aerospace and is subject to intellectual property restrictions. It is therefore not included in this repository.

For technical discussions about the methodology, architecture, or approach, feel free to reach out via GitHub.
