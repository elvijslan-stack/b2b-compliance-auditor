# 📊 Autonomous Due Diligence Analyst — LangGraph & pgvector

<div align="center">

### Autonomous Multi-Agent Financial & Legal Due Diligence Engine

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111+-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![LangGraph](https://img.shields.io/badge/Orchestration-LangGraph%200.2+-FF6F00?style=flat-square&logoColor=white)](https://github.com/langchain-ai/langgraph)
[![Vector DB](https://img.shields.io/badge/Vector%20DB-PostgreSQL%20%2B%20pgvector-336791?style=flat-square&logo=postgresql&logoColor=white)](https://github.com/pgvector/pgvector)
[![Pub/Sub](https://img.shields.io/badge/Pub%2FSub-Redis%20Cluster%20SSE-DC382D?style=flat-square&logo=redis&logoColor=white)](#-stateless-distributed-architecture)
[![Sandbox](https://img.shields.io/badge/Sandbox-gVisor%20%2F%20MicroVM%20Isolation-2496ED?style=flat-square&logo=docker&logoColor=white)](#-deterministic-financial-engine--sandbox)
[![Compliance](https://img.shields.io/badge/Jurisdictions-BaFin%20%7C%20EUR--Lex%20%7C%20EU--Sanctions-003399?style=flat-square)](https://eur-lex.europa.eu/)

[Executive Summary](#-executive-summary) • [System Architecture](#-end-to-end-system-architecture) • [Agent Topology](#-agent-topology--cognitive-state-machine) • [Financial Sandbox](#-deterministic-financial-engine--sandbox) • [Distributed Scaling](#-stateless-distributed-architecture) • [Engineering & Systems Architecture](#-engineering--systems-architecture)

</div>

---

## 📌 Executive Summary

Manual Due Diligence (DD) across commercial filings (10-K, HGB/IFRS annual reports), market intelligence, and regulatory enforcement databases is plagued by fragmented evidence, cognitive confirmation bias, and high-liability calculation errors. Traditional "naive RAG" systems exacerbate these risks by hallucinating accounting figures and failing to reconcile discrepancies between corporate narratives and external market realities.

The **Autonomous Due Diligence Analyst** is an enterprise-grade multi-agent research platform engineered by **[landmansIT](https://github.com/elvijslan-stack)**. Built on **LangGraph**, **pgvector**, an **isolated gVisor/MicroVM code execution sandbox**, and **Redis Pub/Sub event streaming**, it ingests corporate reports, detects emerging operational and regulatory signals, autonomously rewrites queries for external news verification, and cross-checks competing claims.

Beyond passive summarization, the system functions as a **Deterministic Audit & Decision Engine**:
- **Zero-Hallucination Math:** LLMs are strictly forbidden from performing arithmetic. Metrics (YoY Growth, EBITDA Multiples, DSO, FCF) are evaluated through verified AST-checked formulas or executed inside an air-gapped container with zero network access.
- **Automated Contradiction Arbitrage:** Claims are extracted, typed, and reconciled using relative tolerance thresholds ($\Delta \ge 5\%$) weighted against an authoritative source credibility hierarchy.
- **Exhaustive Evidence Traceability:** Every line of the compiled **Investment Memo** is cryptographically anchored to verbatim citations, exact page coordinates, and publication metadata.

---

## 🖥 Modern Analyst Dashboard

<div align="center">
  <img src="https://raw.githubusercontent.com/elvijslan-stack/due-diligence-analyst/main/docs/dashboard.png" alt="Autonomous Due Diligence Analyst Dashboard" width="900" style="border-radius: 8px; box-shadow: 0 4px 20px rgba(0,0,0,0.5);">
  <p><em>Real-time audit dashboard with Server-Sent Events (SSE), background job history, and Human-in-the-Loop approval gates.</em></p>
</div>

---

## 🏛 End-to-End System Architecture

The pipeline processes targets through an asynchronous, budget-controlled cognitive loop, coupling layout-aware ingestion, dense/lexical retrieval, deterministic sandbox execution, and human oversight:

```mermaid
flowchart TD
    Start(["Target Ingestion: Company & Hypothesis"])
    Parser["Layout-Aware Parser: Reports, 10-K & Tables"]
    Planner["Planner Agent: Sub-Question Decomposition"]

    Start --> Parser
    Parser --> Planner

    subgraph RAGSubsystem ["⚡ Bimodal RAG & Signal Extraction Pipeline"]
        direction TB
        HybridRAG["Hybrid Retrieval: pgvector + BM25"]
        Reranker["Cross-Encoder: BAAI/bge-reranker-base"]
        SigDetector["Signal Detector: Operational, Legal & Market Markers"]

        HybridRAG --> Reranker
        Reranker --> SigDetector
    end

    Planner --> HybridRAG

    subgraph DiscoveryLoop ["🌐 External Discovery & Sanitization Loop"]
        direction TB
        QueryRewriter["Query Rewriter: Contextual Discovery Queries"]
        WebAgent["Web & Registry Agent: BaFin, EUR-Lex & News APIs"]
        PromptGuard["Prompt Guard: Anti-Injection & Exfiltration Sanitizer"]

        QueryRewriter --> WebAgent
        WebAgent --> PromptGuard
    end

    SigDetector -->|"Follow-up Required"| QueryRewriter

    subgraph VerificationEngine ["🛡️ Deterministic Verification Engine"]
        direction TB
        ClaimExtractor["Claim Extractor: Tables, Numbers & Statements"]
        Contradiction["Contradiction Verifier: Credibility & Tolerance Arbitrage"]
        FinanceEngine["Deterministic Finance Engine: Air-Gapped Sandbox"]
        LegalRiskLinker["Legal Risk Linker: Severity to Score Mapping"]

        ClaimExtractor --> Contradiction
        Contradiction --> FinanceEngine
        FinanceEngine --> LegalRiskLinker
    end

    SigDetector -->|"Direct Evidence"| ClaimExtractor
    PromptGuard -->|"Sanitized Evidence"| ClaimExtractor

    subgraph SynthesisPhase ["📋 Governance & Memo Synthesis"]
        direction TB
        RiskScorer["Risk Matrix Synthesizer: Multi-Factor Scoring"]
        HITLGate{"HITL Approval Gate: Human Oversight"}
        MemoWriter["Report Writer: Sourced Investment Memo"]
        ExportEngine["Export Engine: ReportLab PDF / Markdown / JSON"]
        Terminate(["Audit Terminated"])

        RiskScorer --> HITLGate
        HITLGate -->|"Approved"| MemoWriter
        HITLGate -->|"Rejected"| Terminate
        MemoWriter --> ExportEngine
    end

    LegalRiskLinker --> RiskScorer
```

---

## 🤖 Agent Topology & Cognitive State Machine

The orchestration layer models the audit workflow as an explicit state machine over `DDState`. Agents operate strictly within bounded operational boundaries:

| Agent | Responsibility | Analytical Output |
| :--- | :--- | :--- |
| **`PlannerAgent`** | Deconstructs investment hypotheses into prioritized, verifiable sub-questions. | `ResearchPlan` (Sub-queries + rationales) |
| **`SignalDetectorAgent`** | Scans semantic chunks for stress indicators (supply chains, covenant breaches, ESG). | `Signal[]` (Categorized by severity) |
| **`QueryRewriterAgent`** | Transforms internal signal alerts into external discovery queries for real-time verification. | `FollowUpQuery[]` (Domain-targeted) |
| **`WebSearchAgent`** | Retrieves real-time regulatory filings and news; filtered via `PromptGuard`. | `Evidence[]` (External provenance) |
| **`ClaimExtractorAgent`** | Isolates numerical claims and factual assertions; preserves table topologies without fragmenting. | `Claim[]` (Values, units, bounding contexts) |
| **`ContradictionAgent`** | Detects metric discrepancies across disparate sources; applies credibility rank resolution. | `Contradiction[]` (Severity & auto-resolution) |
| **`FinanceCalcAgent`** | Executes math through native deterministic libraries or sandboxed subprocesses. | `Calculation[]` (Formula + exact audit trail) |
| **`RegulatoryAgent`** | Mines EUR-Lex, BaFin, and EU sanctions lists for active proceedings and obligations. | `LegalFinding[]` (Jurisdiction-bound) |
| **`RiskScorerAgent`** | Synthesizes multidimensional risk matrices with dynamic penalty scoring. | `RiskMatrix` (Overall score + confidence index) |
| **`ReportWriterAgent`** | Drafts comprehensive Investment Memos with strict quote attribution and gap disclosures. | `InvestmentMemo` (Cited decision baseline) |

---

## 📐 Deterministic Financial Engine & Sandbox

To guarantee institutional compliance, this system eliminates language model arithmetic. Calculations are processed through two deterministic paths:

### 1. Hardened Metric Registry (`src/dd_analyst/finance/metrics.py`)
Pre-compiled, unit-tested accounting formulas executing without LLM inference:
- **Profitability:** Gross Margin, Operating Margin, EBITDA Margin
- **Cash Flow Dynamics:** Free Cash Flow ($FCF = OCF - |CapEx|$)
- **Capital Structure:** Net Debt ($Total Debt - Cash$), Net Debt / EBITDA
- **Working Capital:** Days Sales Outstanding ($DSO = \frac{A/R}{Revenue} \times 365$), Current Ratio

### 2. Air-Gapped Code Interpreter & MicroVM Sandbox (`src/dd_analyst/sandbox/interpreter.py`)
When custom metrics are required, LLM-generated code is executed inside an isolated runsc/gVisor environment:
- **Process Isolation:** Invoked via `python -I -S -c` (isolated mode, ignoring system `PYTHONPATH`, user site-packages, and environment variables).
- **Socket Nullification:** Network interfaces are intercepted and neutralized at the Python runtime level; any socket instantiation throws an immediate runtime exception.
- **Resource Constraints:** Governed by strict timeouts (`DD_SANDBOX_TIMEOUT_S`) and capped buffer outputs (`DD_SANDBOX_MAX_OUTPUT_CHARS`) within ephemeral directories.

---

## 📡 Stateless Distributed Architecture

Engineered for horizontal scalability across multi-replica Kubernetes clusters:

```text
 Client (Browser)
       │
       ▼ (HTTP POST /analyze)
 [FastAPI Pod A] ──► Pushes Job to Queue / Worker Pool
                           │
                           ▼
                 [LangGraph Analysis Runner]
                           │
                           ▼ (Emits ProgressEvent)
                 [Redis Pub/Sub Channel: dd:jobs:{job_id}]
                           │
       ┌───────────────────┴───────────────────┐
       ▼                                       ▼
 [FastAPI Pod A]                         [FastAPI Pod B]
       │ (SSE Stream)                          │ (SSE Stream)
       ▼                                       ▼
 Client Session A                        Client Session B
```

- **Stateless Web-Tier:** Eliminates sticky session dependencies. Any cluster pod can serve Server-Sent Events (SSE) while another pod executes the analysis loop.
- **High-Throughput Concurrency:** Redis Pub/Sub acts as an asynchronous event bus for distributed job tracking, state hydration, and multi-tenant rate limiting.

---

## 🔍 Bimodal RAG & Reciprocal Rank Fusion

The ingestion and retrieval subsystem (`src/dd_analyst/rag/`) bridges high-density vector representations with exact lexical keyword matching:

```text
User Query
    │
    ├──► Dense Embedding ──────► pgvector Store (Cosine Distance, IVFFlat / HNSW)
    │                               │
    └──► Lexical Tokenizer ────► Native Okapi BM25 Index (IDF Term Saturation)
                                    │
                                    ▼
                         Weighted RRF Fusion:
                       Score = Σ [ w_i / (k + rank_i) ]
                                    │
                                    ▼
                     Cross-Encoder Reranking
                 (BAAI/bge-reranker-base + Sigmoid Normalization)
```

1. **pgvector Storage:** Chunks are vectorized using Hashing (zero-dependency offline mode), Ollama (`nomic-embed-text`), or OpenAI embeddings into PostgreSQL.
2. **Native Okapi BM25:** Implemented without external binary dependencies (`SimpleBM25`), providing exact matching on German/English accounting terminology.
3. **Reciprocal Rank Fusion (RRF):** Fuses ranking distributions with configurable weights (`DD_HYBRID_VECTOR_WEIGHT=0.6`, `DD_HYBRID_BM25_WEIGHT=0.4`, $k=60$).
4. **Cross-Encoder Reranker:** Top candidates pass through a secondary neural ranker (`bge-reranker-base`) with dynamic logit-to-sigmoid transformation and lexical fallback.

---

## 🛡️ Defense-in-Depth & Security Controls

Operating on untrusted external news feeds introduces prompt injection and data exfiltration vectors. The analyst enforces a multi-tier defense architecture:

- **PromptGuard Ingestion Filter (`security/prompt_guard.py`):** Automatically neutralizes adversarial control codes, strips non-printable Unicode ranges (such as `U+E0000–U+E007F`), and redacts prompt hijack attempts (`ignore previous instructions`, `system:`, exfiltration probes via `curl/wget`).
- **Granular Token & Budget Trackers (`cost_limits.py`):** Protects compute resources by terminating jobs exceeding token ceilings, web search allowances, or recursion iteration boundaries.
- **Audit Trails:** Every state transition, authentication event, HITL decision, and calculation is recorded in append-only JSONL files or dedicated PostgreSQL audit tables (`dd_audit_events`).

---

## 🚀 Quickstart & Production Setup

### Installation

```bash
# Clone the repository
git clone https://github.com/elvijslan-stack/due-diligence-analyst.git
cd due-diligence-analyst

# Initialize virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install all enterprise dependencies
pip install -e ".[all]"

# Configure runtime environment
cp .env.example .env
```

### CLI Execution

Execute an end-to-end audit directly against local corporate filings:

```bash
# Analyze a target company against a local corpus
dd-analyst analyze \
  --company "Acme Corp" \
  --corpus ./corpus/acme_2024 \
  --question "Evaluate liquidity, Asian supply chain dependencies, and pending antitrust risks."

# Output audit findings directly to structured JSON
dd-analyst analyze --company "Acme Corp" --corpus ./corpus --json
```

### API & Modern WebUI

Launch the asynchronous FastAPI backend with embedded Server-Sent Events (SSE) and interactive browser UI:

```bash
dd-analyst serve --host 0.0.0.0 --port 8000
```
- **Web UI:** Navigate to `http://localhost:8000/`
- **Swagger Documentation:** Available at `http://localhost:8000/docs`
- **Prometheus Metrics:** Scraped at `http://localhost:8000/prometheus`

---

## 🧑‍💻 Engineering & Systems Architecture

Architected by **Elvijs Landmans ([landmansIT](https://github.com/elvijslan-stack))**.

- **Neuro-Symbolic Due Diligence:** Enforcing a rigorous separation between probabilistic LLM reasoning (hypothesis generation, document summarization) and deterministic symbolic compute (financial metric formulas, discrepancy arbitration).
- **Cyclic State Graph with Budgetary Gating:** Encapsulating execution in LangGraph with persistent memory checkpointing, bounded loops, and strict cost limits to eliminate infinite recursion in high-latency discovery workflows.
- **Zero-Trust MicroVM Isolation:** Nullifying host-kernel exploits by running unverified code in air-gapped gVisor/subprocess sandboxes stripped of networking privileges, file paths, and environment access.
- **Stateless Cloud-Native Scale:** Eliminating single-point-of-failure bottlenecks via a distributed Redis Pub/Sub event bus, decoupling the compute-heavy agent runners from the SSE streaming web tier.
- **Authoritative Provenance Graphs:** Enforcing strict citation verification where every claim is validated against document IDs, page offsets, and publication dates—retaining an auditable link between the final investment memo and source filings.

---

## ⚖️ Legal & Financial Disclaimer

> **CRITICAL NOTICE:** This platform is an automated decision-support research aggregator designed for corporate finance and legal analysts. **It does NOT offer financial, investment, accounting, or legal advice.** All synthesized investment memos, risk matrices, and calculated financial ratios must be independently reviewed and verified by qualified professionals prior to capital allocation or contractual commitment.

