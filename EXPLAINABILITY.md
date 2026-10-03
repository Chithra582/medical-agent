# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Medical Agent** (`medical-agent`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Medical Agent (`medical-agent`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Life Sciences / Biomedical AI, Genomics & Computational Biology  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

The agent executes biomedical research and computational biology tasks through a deterministic, 5-stage orchestration pipeline.

### 1. Decision Architecture

The runtime intake, state classification, evaluation, and execution tracking operate across a deterministic, five-stage pipeline:

```
+-----------------------------------------------------------------------------------+
|                        Deterministic Biomedical Pipeline                          |
+-----------------------------------------------------------------------------------+
|  [Stage 1: Scientific Query Ingestion & Target Entity Resolution]                 |
|     --> Extract gene symbols, SMILES structures, cell types, or protocol queries  |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 2: Biosecurity Verification & Governance Gate]                            |
|     --> Screen targets against pathogen watchlists, toxin databases, & PHI rules  |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 3: Multi-Modal Biomedical Tool Dispatch]                                  |
|     --> Route to CRISPR design, ADMET predictors, scRNA annotators, or PubMed     |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 4: Quantitative In Silico Hypothesis Scoring]                            |
|     --> Evaluate binding affinities, off-target risks, ADMET ranges, and logFC     |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|  [Stage 5: Evidence Synthesis & Laboratory Protocol Formulation]                  |
|     --> Generate reproducible bench protocols, literature citations, and warnings |
+-----------------------------------------------------------------------------------+
```

### 2. Decision Logic & Routing Formulations

Candidate perturbation targets and molecular designs are scored via a multi-objective bio-affinity formulation:

$$S_{\text{bio}}(x) = w_1 \cdot \text{TargetAffinity}(x) + w_2 \cdot (1 - \text{OffTargetRisk}(x)) + w_3 \cdot \text{Bioavailability}(x) - w_4 \cdot \text{Toxicity}(x)$$

Where:
- $w_1 = 0.40$: Primary on-target binding/knockout efficiency.
- $w_2 = 0.25$: Genome-wide specificity and off-target avoidance penalty.
- $w_3 = 0.20$: Pharmacokinetic drug-likeness (Lipinski Rule of Five compliance).
- $w_4 = 0.15$: In silico predicted hepatotoxicity or cellular cytotoxicity.

Biomedical tool selection probability for research task $T$ is computed as:

$$P(\text{Tool}_i \mid T) = \frac{\exp(\mathbf{u}_i^{\top} \mathbf{v}_T)}{\sum_{j=1}^{K} \exp(\mathbf{u}_j^{\top} \mathbf{v}_T)}$$

Where $\mathbf{v}_T$ is the query semantic embedding and $\mathbf{u}_i$ denotes the capability vector of tool $i$.

### 3. Thresholding & Refusal Decision Criteria

Medical Agent enforces strict operational boundaries and deterministic refusal thresholds:
- **Refusal on ERR_BIOSECURITY_HAZARD**: Select Agent / Pathogen Match (Sequence / Entity $> 0.85$ match) halts execution with code `ERR_BIOSECURITY_HAZARD`.
- **Refusal on ERR_NON_CLINICAL_SCOPE**: Direct Clinical Prescribing Request (Intent = Treatment Prescription) halts execution with code `ERR_NON_CLINICAL_SCOPE`.
- **Refusal on ERR_INVALID_SMILES_STRING**: Invalid Chemical SMILES Syntax (RDKit parse failure) halts execution with code `ERR_INVALID_SMILES_STRING`.
- **Refusal on ERR_UNKNOWN_BIOMARKER**: Unresolvable Gene / Protein ID (NCBI / UniProt lookup empty) halts execution with code `ERR_UNKNOWN_BIOMARKER`.
- **Refusal on ERR_PHI_DETECTED**: Protected Health Info (PHI) Ingestion (Regex match on SSN/MRN/DOB) halts execution with code `ERR_PHI_DETECTED`.

### 4. Fallback Decision Mechanism

Continuous operational stability is maintained through layered fault recovery:
- **Tier 1 (Algorithmic Fallback)**: If a specialized highthroughput tool fails (e.g., deeplearning structural folding), fallback to statistical heuristics or homology modeling.
- **Tier 2 (External Knowledge Base Fallback)**: If local datalake embeddings lack specific target information, trigger automated NCBI Entrez or EuropePMC live API queries.
- **Model Fallback Cascade**: High-level reasoning and synthesis default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 5. Human-in-the-Loop Governance

Human operators retain sovereign authority over the multi-agent execution lifecycle:
- **Tier 3 (Human Principal Investigator Verification)**: Any proposed experimental protocol modifying highrisk cellular assays or geneediting vectors mandates manual signoff by a qualified principal investigator.
- **Benchmark Trajectory Auditing**: Operators inspect evaluation traces, raw generation tokens, and container logs to verify scoring fidelity.

---

## The Data It Uses

Medical Agent operates under strict principles of data minimization, environment isolation, and privacy protection.

### 1. Ingested Input Data

The framework processes only operational data necessary to perform its functions:
- **Molecular Representations**: SMILES strings, FASTA sequences, PDB atomic coordinates.
- **Transcriptomic Matrices**: Single-cell RNA-seq count tables (`.h5ad`, 10x Genomics matrices).
- **Experimental Queries**: Free-text natural language hypothesis descriptions and target criteria.

### 2. Configuration & Reference Data

- **Genomic & Proteomic Databases**: UniProtKB, Ensembl, HGNC, NCBI RefSeq, PDB.
- **Pharmacological Resources**: ChEMBL, DrugBank, PubChem, BindingDB, Tox21.
- **Scientific Literature**: PubMed Central, BioRxiv preprints, OpenAlex citation index.

### 3. Base Model & Inference Lineage

- **Inference Models**: Claude 3.5 Sonnet / Claude 4, GPT-4o, and specialized biomedical embedders.
- **Computational Libraries**: RDKit, BioPython, Scanpy, AnnData, SciPy, PyTorch.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against indirect prompt injection, credential leakage, and unauthorized external API dispatch.
- **Local Environment Isolation**: Agent execution workspaces, intermediate scratchpads, and vector stores reside strictly within designated local project directories.
- **Automated Secret Scrubbing**: API keys, database credentials, and personal credentials are automatically redacted prior to embedding or logging.
- **Zero Commercial Monetization**: Prompts, intermediate reasoning trajectories, and task deliverables are never commercialized or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of Medical Agent is essential for effective deployment.

### 1. In Silico Versus In Vitro Validation Gap
- **Limitation**: Computational affinity predictions and perturbation rankings do not guarantee functional wet-lab replication.
- **Mitigation**: Flag all outputs with experimental confidence scores and recommend pilot validation assays with positive/negative controls.

### 2. Heavy Computational & Storage Requirements
- **Limitation**: Initializing the comprehensive biomedical data lake requires ~11GB of disk storage and multi-core memory allocations.
- **Mitigation**: Provide modular datalake streaming and lightweight mode bypassing local file downloads for literature-only tasks.

### 3. Context Length Bottlenecks in Single-Cell Datasets
- **Limitation**: High-dimensional single-cell matrices containing tens of thousands of genes cannot fit directly into LLM context windows.
- **Mitigation**: Perform dimensionality reduction (PCA, UMAP) and pass high-variance marker gene summaries rather than raw count matrices.

### 4. Ambiguity in Legacy Gene Synonymy
- **Limitation**: Historic literature frequently uses obsolete gene or protein aliases, introducing resolution ambiguities.
- **Mitigation**: Normalize all biological entity names through HGNC and NCBI Gene canonical lookup tables before processing.

### 5. Potential Hallucination in Novel Chemical Entities
- **Limitation**: Generative language models may propose non-synthesizable chemical molecules with valence violations.
- **Mitigation**: Route all proposed molecules through RDKit molecular sanitization and synthetic accessibility score (SAScore) filters.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & routing formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested input data & query streams | Section 1 | Verified |
| - Configuration & reference schemas | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - In Silico Versus In Vitro Validation Gap | Section 1 | Verified |
| - Heavy Computational & Storage Requirements | Section 2 | Verified |
| - Context Length Bottlenecks in Single-Cell Datasets | Section 3 | Verified |
| - Ambiguity in Legacy Gene Synonymy | Section 4 | Verified |
| - Potential Hallucination in Novel Chemical Entities | Section 5 | Verified |
