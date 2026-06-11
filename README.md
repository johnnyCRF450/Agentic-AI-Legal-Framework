# Agentic-AI-Legal-Framework
multi-agent system to assist military lawyers by automating data-intensive tasks and augmenting their legal reasoning capabilities 

╔══════════════════════════════════════════════════════════════════════════════╗
║          PROJECT MEDUSA — README                                            ║
║          Multi-Agent Legal Assistant for Military Justice (UCMJ/MCM)       ║
╚══════════════════════════════════════════════════════════════════════════════╝

OVERVIEW
────────
Project Medusa is a reference implementation of a multi-agent AI legal assistant
tailored for military lawyers operating under the Uniform Code of Military Justice
(UCMJ) and the Manual for Courts-Martial (MCM). It demonstrates a full agentic
pipeline — from secure document ingestion to syllogistic legal drafting — using
only the Python standard library (no pip installs required for the core engine).

QUICK START
───────────
  # Text dashboard (Envision / plain Python):
  python project_medusa.py

  # Streamlit web UI (requires: pip install streamlit):
  streamlit run project_medusa.py

  # Plotly Dash web UI (requires: pip install dash):
  python -c "from project_medusa import build_dash_app; build_dash_app().run(debug=True)"

ARCHITECTURE
────────────
  ┌─────────────────────────────────────────────────────────┐
  │                        USER                             │
  └────────────────────────┬────────────────────────────────┘
                           │ query + document
  ┌────────────────────────▼────────────────────────────────┐
  │              ORCHESTRATOR (Sec 3.1)                     │
  │   Router ──► Central Planner  ──or──  Event-Driven      │
  │              (Supervisor)             (Swarm/Blackboard) │
  └──┬──────┬──────┬──────┬──────┬──────┬──────┬───────────┘
     │      │      │      │      │      │      │
  EvidGov  GovCo  Intel  Risk  Alloc  Res   Draft
  Agent    Agent  Agent  Agent Agent  Agent  Agent
     │                    │            │      │
  PseudoKG             XGBoost      GraphRAG Claude
  (Sec 4)              +SHAP        (Sec 5.1)(Sec 2.1)
     │                                        │
  Provenance Ledger ◄─────────────────────────┘
  (hash-chained WORM, Sec 6.1)

KEY COMPONENTS
──────────────
  ClaudeClient        Stub for Anthropic Claude (swap in real API key)
  LegalBERTEmbedder   Stub for LegalBERT embeddings (64-dim deterministic)
  KnowledgeGraph      IRAC-schema KG with graph traversal (issue→rule→source)
  VectorStore         Cosine-similarity semantic search with metadata filtering
  PseudonymizationPipeline  Regex+NER PII detection, envelope encryption, KMS
  GuardrailStack      4-level safety: prompt / retrieval / output / action
  GraphRAG            Two-pronged retrieval: KG traversal + vector search
  SyllogismReasoner   Legal Syllogism Prompting (major/minor premise + CoT)
  RiskPredictionAgent XGBoost-emulated with SHAP attributions + fairness check
  ResourceAllocationAgent  Constraint-optimization heuristic for team planning
  ProvenanceLedger    SHA-256 hash-chained append-only audit log (WORM-style)
  FeedbackStore       RLHF loop: pairwise / scalar / structured critique → PPO reward
  Orchestrator        Hybrid supervisor+swarm with dynamic Router-Solver switching

SECURITY MODEL (Sec 4)
───────────────────────
  • PII detected via regex + NER stub → replaced with [TYPE-HEXTOKEN]
  • Each PII value encrypted with a unique DEK under envelope encryption
  • DEK encrypted by emulated hardware KMS (swap in AWS KMS / Azure Key Vault)
  • Mapping DB: {CaseID, Token, EncryptedValue, EncryptedDEK}
  • De-anonymization requires: role=CaseLead AND assigned_cases∋CaseID AND MFA

PROVENANCE & AUDITABILITY (Sec 6.1)
─────────────────────────────────────
  • Every agent action appended to an immutable hash-chained ledger
  • Each entry stores: actor, action, timestamp, rationale, SHA-256 of I/O
  • verify_integrity() walks the chain — any tamper breaks the hash sequence
  • trace(correlation_id) retrieves all events for a single query end-to-end

RLHF FEEDBACK LOOP (Sec 5.2)
──────────────────────────────
  • Lawyers submit scalar ratings (1-5) across Legal Accuracy, Citation Quality,
    Clarity, Completeness + optional structured critique category
  • Hybrid reward: R = 0.7*R_accuracy + 0.3*R_utility − 0.3*(critique≠NONE)
  • PPO-style targeted update: reward applied only to the responsible agent

GUARDRAILS (Sec 6.2)
─────────────────────
  L1  Prompt Guardrail    — blocks prompt injection patterns on input
  L2  Retrieval Guardrail — restricts to authorized corpora; strips injections
  L3  Output Guardrail    — scans for raw PII leakage; appends legal disclaimer
  L4  Action Guardrail    — validates RBAC role before any privileged action

PRODUCTION SWAP-INS
────────────────────
  Stub                  → Production replacement
  ─────────────────────────────────────────────
  ClaudeClient          → anthropic.Anthropic().messages.create(...)
  LegalBERTEmbedder     → sentence-transformers / LegalBERT HuggingFace model
  EmulatedKMS           → boto3 KMS / azure.keyvault.keys
  SecureMappingStore    → boto3 DynamoDB / Azure Cosmos DB
  KnowledgeGraph        → Neo4j / Amazon Neptune
  VectorStore           → Pinecone / Weaviate / pgvector
  PII detection regex   → Microsoft Presidio / AWS Macie
  ProvenanceLedger      → AWS S3 + Object Lock (WORM) / blockchain ledger

DISCLAIMER
──────────
  All outputs are AI-generated decision-support tools. They do NOT constitute
  legal advice. A qualified judge advocate must review and make all final
  decisions. This system is designed to augment — not replace — human judgment.

VERSION
───────
  Project Medusa v1.0.0  |  Architecture: Sec 2–6 (see inline comments)
  Python stdlib only for core engine. Streamlit/Dash optional for web UI.



████████████████████████████████████████████████████████████████████████████
█                 █  PROJECT MEDUSA — ARCHITECTURE DIAGRAM                  
████████████████████████████████████████████████████████████████████████████
┌──────────────────────────────────────────────────────────────────────────┐
│                  ◈  A — SYSTEM OVERVIEW & DATA FLOW  ◈                   │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                     ┌──────────┐    query + document                     │
│   │   USER   │ ──────────────────────────────────────────────────────►   │
│  │ (JA /    │                                                        │   │
│   │paralegal)│ ◄───────────────────────────── final report ──────────┤   │
│  └──────────┘                                                        │   │
│                                                                      ▼   │
│            ┌─────────────────────────────────────────────────────────┐   │
│             │            ORCHESTRATOR  (Sec 3.1)                     │   │
│             │  ┌──────────┐   classifies query                       │   │
│             │  │  Router  │ ─────────────────────────────────────┐   │   │
│            │  └──────────┘                                       │   │   │
│            │       │ procedural                  ambiguous       │   │   │
│            │       ▼                                 ▼           │   │   │
│             │  ┌──────────┐                   ┌────────────┐     │   │   │
│             │  │ Central  │                   │   Event-   │     │   │   │
│             │  │ Planner  │                   │   Driven   │     │   │   │
│             │  │(Supervisor│                  │   Swarm    │     │   │   │
│             │  └────┬─────┘                   └─────┬──────┘     │   │   │
│             └───────┼──────────────────────────────┼─────────────┘   │   │
│                    │                              │                  │   │
│      ┌──────────────┼──────────────────────────────┼──────────────┐  │   │
│     │              ▼    AGENT LAYER (Sec 3.2)      ▼              │  │   │
│      │  ┌─────────┐ ┌──────────┐ ┌────────┐ ┌──────────────────┐ │  │    │
│      │  │Evidence │ │Governance│ │Stake-  │ │Risk  │ Resource   │ │  │   │
│      │  │Collect. │ │Design /  │ │holder  │ │Pred. │ Allocation │ │  │   │
│      │  │Agent    │ │Coord.    │ │Intel   │ │Agent │ Agent      │ │  │   │
│      │  └────┬────┘ └────┬─────┘ └───┬────┘ └──┬───┴─────┬──────┘ │  │   │
│     │       │           │           │          │         │        │  │   │
│     │  ┌────▼───────────▼───────────▼──────────▼─────────▼──────┐ │  │   │
│     │  │     Legal Research Agent  ──►  Legal Drafting Agent     │ │  │  │
│     │  │     (GraphRAG, Sec 5.1)        (Claude/Syllogism)       │ │  │  │
│      │  └───────────────────────────────────────────────────────┘ │  │   │
│     │                    BLACKBOARD (shared state)                │  │   │
│     └─────────────────────────────────────────────────────────────┘  │   │
│                                                                       │  │
│     ┌──────────────────────────────────────────────────────────────┐ │   │
│      │  PROVENANCE LEDGER  (hash-chained WORM, Sec 6.1)            │◄┘   │
│       │  GUARDRAIL STACK    (L1 prompt / L2 retrieval /             │    │
│       │                      L3 output / L4 action, Sec 6.2)        │    │
│      └──────────────────────────────────────────────────────────────┘    │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────────┐
│             ◈  B — PII PSEUDONYMIZATION PIPELINE (Sec 4)  ◈              │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                        Raw Document (PDF/DOCX/TXT)                       │
│                                        │                                 │
│                                        ▼                                 │
│       ┌───────────────────────────────────────────────────────────┐      │
│       │ STAGE 1: Secure Ingestion                                 │      │
│        │  • AES-256 at-rest  •  TLS 1.2+ in-transit               │      │
│       │  • SHA-256 document hash logged to Provenance Ledger      │      │
│       └───────────────────────────┬───────────────────────────────┘      │
│                                                   │                      │
│                                                   ▼                      │
│       ┌───────────────────────────────────────────────────────────┐      │
│       │ STAGE 2: PII Detection & Pseudonymization                 │      │
│        │  ┌─────────────────────┐   ┌──────────────────────────┐  │      │
│        │  │ Regex patterns      │   │ NER (LegalBERT / Presidio│  │      │
│        │  │  SSN / EMAIL / PHONE│   │  PERSON / RANK_NAME)     │  │      │
│        │  └──────────┬──────────┘   └───────────┬──────────────┘  │      │
│       │             └──────────────┬────────────┘                 │      │
│       │                           ▼                               │      │
│       │           Coreference Resolution (same entity → same tok) │      │
│       │                           │                               │      │
│       │                           ▼                               │      │
│       │         [TYPE-HEXTOKEN]  generated per PII value          │      │
│       └───────────────────────────┬───────────────────────────────┘      │
│                                                   │                      │
│                                                   ▼                      │
│       ┌───────────────────────────────────────────────────────────┐      │
│       │ STAGE 3: Envelope Encryption (Sec 4.4)                    │      │
│       │                                                           │      │
│        │   ┌──────┐  generate DEK   ┌─────────────────────────┐   │      │
│       │   │ KMS  │ ──────────────► │ plaintext DEK (transient │   │      │
│       │   │(HSM) │                 │ — discarded immediately) │   │      │
│        │   └──┬───┘                 └────────────┬────────────┘   │      │
│        │      │ encrypted DEK                    │ encrypt PII    │      │
│        │      ▼                                  ▼                │      │
│        │   {CaseID, Token, EncryptedPII, EncryptedDEK}            │      │
│        │         stored in Secure Mapping DB                      │      │
│       └───────────────────────────┬───────────────────────────────┘      │
│                                                   │                      │
│                                                   ▼                      │
│       ┌───────────────────────────────────────────────────────────┐      │
│       │ STAGE 4: De-anonymization API (Sec 4.5)                   │      │
│        │  OAuth2/OIDC + MFA  →  RBAC (role=CaseLead)              │      │
│       │                     →  ABAC (assigned_cases ∋ CaseID)     │      │
│        │  Authorized: KMS decrypts DEK in-memory → returns PII    │      │
│       │  Unauthorized: PermissionError raised immediately         │      │
│       └───────────────────────────────────────────────────────────┘      │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────────┐
│         ◈  C — GRAPHRAG + LEGAL SYLLOGISM REASONING (Sec 5.1)  ◈         │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                                 User Query                               │
│                                       │                                  │
│                                       ▼                                  │
│        ┌──────────────────────────────────────────────────────────┐      │
│        │ Legal Research Agent — Query Decomposition               │      │
│        │   Identifies: LegalIssue node (e.g., issue:theft)        │      │
│        └────────────────────┬─────────────────────────────────────┘      │
│                                                │                         │
│                          ┌─────────────┴──────────────┐                  │
│                          ▼                            ▼                  │
│              ┌─────────────────┐       ┌────────────────────┐            │
│              │  KG TRAVERSAL   │       │  VECTOR SEARCH     │            │
│              │  (Sec 2.2)      │       │  (Sec 2.2)         │            │
│              │                 │       │                    │            │
│              │ issue:theft     │       │ embed(query)       │            │
│              │   │ applies     │       │ cosine sim.        │            │
│              │   ▼             │       │ metadata filter    │            │
│              │ rule:larceny    │       │ {source: UCMJ}     │            │
│              │   │ cites       │       │                    │            │
│              │   ▼             │       │ top-k chunks       │            │
│              │ src:ucmj-121   │       │ returned           │             │
│              └────────┬────────┘       └────────┬───────────┘            │
│                             │                         │                  │
│                             └────────────┬────────────┘                  │
│                                                ▼                         │
│        ┌──────────────────────────────────────────────────────────┐      │
│        │ GUARDRAIL L2: corpus allowlist + injection strip         │      │
│        └────────────────────────────┬─────────────────────────────┘      │
│                                                    │                     │
│                                                    ▼                     │
│        ┌──────────────────────────────────────────────────────────┐      │
│       │ SYLLOGISM PROMPT (Legal Drafting Agent + Claude)          │      │
│        │                                                          │      │
│        │  MAJOR PREMISE : rule:larceny (from KG traversal)        │      │
│        │  GOVERNING SRC : UCMJ Art. 121 (from KG cites edge)      │      │
│       │  PRECEDENTS    : top-k semantic chunks (from VectorStore) │      │
│       │  MINOR PREMISE : pseudonymized case facts                 │      │
│       │                  ↓                                        │      │
│       │  CONCLUSION    : Claude chain-of-thought judgment         │      │
│        └────────────────────────────┬─────────────────────────────┘      │
│                                                    │                     │
│                                                    ▼                     │
│        ┌──────────────────────────────────────────────────────────┐      │
│        │ GUARDRAIL L3: PII scan + legal disclaimer appended       │      │
│        └──────────────────────────────────────────────────────────┘      │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────────┐
│         ◈  D — PROVENANCE LEDGER (hash-chained WORM, Sec 6.1)  ◈         │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                                  GENESIS                                 │
│                                      │                                   │
│                                      ▼                                   │
│        ┌─────────────────────────────────────────────────────────┐       │
│        │ Event 1: Orchestrator / route_query                     │       │
│        │   actor=Orchestrator  action=route_query                │       │
│        │   rationale='Classified as central_planner'             │       │
│        │   prev_hash=GENESIS   entry_hash=SHA256(this_event)     │       │
│        └────────────────────────────────┬────────────────────────┘       │
│                                                 │ prev_hash              │
│                                                      ▼                   │
│        ┌─────────────────────────────────────────────────────────┐       │
│        │ Event 2: EvidenceCollectionAgent / ingest_document      │       │
│        │   payload_hash=SHA256(raw_document)                     │       │
│        │   outcome_hash=SHA256({doc_hash, tokens, ...})          │       │
│        │   prev_hash=hash(Event 1)                               │       │
│        └────────────────────────────────┬────────────────────────┘       │
│                                                      │                   │
│                                                      ▼                   │
│        ┌─────────────────────────────────────────────────────────┐       │
│        │ Event N: LegalDraftingAgent / llm_reasoning_trace       │       │
│        │   Full prompt context logged + CoT output hash          │       │
│        │   prev_hash=hash(Event N-1)                             │       │
│        └────────────────────────────────┬────────────────────────┘       │
│                                                      │                   │
│                                                      ▼                   │
│         verify_integrity() walks chain: prev_hash[i] == hash(event[i-1]) │
│                 Any tamper breaks the sequence → integrity=False         │
│                 trace(correlation_id) → all events for one query         │
│                                                                          │
│         Backing store options (production):                              │
│           • AWS S3 + Object Lock (WORM)                                  │
│           • AWS CloudWatch Logs + Kinesis                                │
│           • Blockchain / distributed ledger                              │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────────┐
│                  ◈  E — RLHF FEEDBACK LOOP (Sec 5.2)  ◈                  │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│      ┌─────────────────────────────────────────────────────────────┐     │
│      │              FEEDBACK COLLECTION (Sec 5.2)                  │     │
│      │                                                             │     │
│      │  Military Lawyer provides:                                  │     │
│      │   • Scalar ratings (1-5): Legal Accuracy, Citation Quality, │     │
│      │     Clarity, Completeness                                   │     │
│       │   • Pairwise comparison: response A vs response B          │     │
│       │   • Structured critique: Incorrect Rule / Flawed Logic /   │     │
│      │     Hallucinated Citation / None                            │     │
│      │   • Corrective edit: direct text correction                 │     │
│      └──────────────────────────────┬──────────────────────────────┘     │
│                                                     │                    │
│                                                     ▼                    │
│      ┌─────────────────────────────────────────────────────────────┐     │
│      │              REWARD MODEL (hybrid)                          │     │
│      │                                                             │     │
│       │  R_accuracy = avg(Legal Accuracy, Citation Quality) / 5    │     │
│       │  R_utility  = avg(Clarity, Completeness) / 5               │     │
│       │  R_total    = 0.7 * R_accuracy + 0.3 * R_utility           │     │
│       │               − 0.3  (if structured critique ≠ NONE)       │     │
│      │                                                             │     │
│      │  Dense token-level penalty: error-flagged sentences         │     │
│      │  receive negative reward at token level                     │     │
│      └──────────────────────────────┬──────────────────────────────┘     │
│                                                     │                    │
│                                                     ▼                    │
│      ┌─────────────────────────────────────────────────────────────┐     │
│      │         PPO UPDATE (Proximal Policy Optimization)           │     │
│      │                                                             │     │
│       │  • Reward applied ONLY to target_agent (no cross-agent     │     │
│      │    contamination)                                           │     │
│       │  • PPO clipping: |ratio - 1| ≤ ε → no catastrophic        │      │
│      │    forgetting of legal knowledge                            │     │
│       │  • Entire feedback event logged to Provenance Ledger       │     │
│      └──────────────────────────────┬──────────────────────────────┘     │
│                                                     │                    │
│                                                     ▼                    │
│                   Improved Agent Policy ──► better future outputs        │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────────┐
│                    ◈  F — AGENT INTERACTION MATRIX  ◈                    │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│          Agents (rows) produce outputs consumed by agents (cols)         │
│           ✓ = direct dependency   · = indirect (via Blackboard)          │
│                                                                          │
│                         Ev  GovD GovC Intel Risk  Alloc Res  Draft       │
│                         ──  ──── ──── ─────  ────  ───── ───  ─────      │
│        EvidenceCollect  ──   ✓    ·    ✓      ✓     ·     ·    ✓         │
│        GovernanceDesign  ·  ──    ✓    ·      ·     ·     ✓    ·         │
│        GovCoordination   ·   ·   ──    ·      ·     ·     ·    ·         │
│        StakeholderIntel  ·   ·    ·   ──      ·     ·     ·    ·         │
│        RiskPrediction    ·   ·    ·    ·      ──    ✓     ·    ·         │
│        ResourceAlloc     ·   ·    ·    ·      ·    ──     ·    ·         │
│        LegalResearch     ·   ·    ·    ·      ·     ·    ──    ✓         │
│        LegalDrafting     ·   ·    ·    ·      ·     ·     ·   ──         │
│                                                                          │
│         All agents → Provenance Ledger (write-only)                      │
│         Orchestrator → Blackboard (shared read/write)                    │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────────┐
│            ◈  G — MULTI-LAYERED GUARDRAIL STACK (Sec 6.2)  ◈             │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│      ┌─────────────────────────────────────────────────────────────┐     │
│      │  L1 — PROMPT GUARDRAIL (Input Filtering)                    │     │
│      │  Detects: prompt injection, out-of-scope queries            │     │
│      │  Pattern: 'ignore previous|disregard.*instructions|...'     │     │
│      │  Action: raise GuardrailViolation → request blocked         │     │
│      └──────────────────────────────┬──────────────────────────────┘     │
│                                              │ clean prompt              │
│                                                     ▼                    │
│      ┌─────────────────────────────────────────────────────────────┐     │
│      │  L2 — RETRIEVAL GUARDRAIL (RAG Protection)                  │     │
│       │  Restricts: only UCMJ / MCM corpus (allowlist)             │     │
│      │  Sanitizes: strips embedded injections from retrieved text  │     │
│      │  Action: drop unauthorized records; redact injections       │     │
│      └──────────────────────────────┬──────────────────────────────┘     │
│                                              │ clean context             │
│                                                     ▼                    │
│      ┌─────────────────────────────────────────────────────────────┐     │
│      │  L3 — OUTPUT GUARDRAIL (Response Filtering)                 │     │
│      │  Scans: raw PII (SSN regex), hallucination signals          │     │
│      │  Appends: mandatory legal disclaimer to every output        │     │
│      │  Action: raise GuardrailViolation on PII leak               │     │
│      └──────────────────────────────┬──────────────────────────────┘     │
│                                              │ safe response             │
│                                                     ▼                    │
│      ┌─────────────────────────────────────────────────────────────┐     │
│      │  L4 — ACTION GUARDRAIL (Tool Governance)                    │     │
│      │  Validates: RBAC role check before any privileged action    │     │
│      │  Prevents: irreversible actions by unauthorized agents      │     │
│      │  Action: raise GuardrailViolation on role mismatch          │     │
│      └──────────────────────────────────────────────────────────────┘    │
│                                                                          │
│       Human-in-the-Loop: all outputs are decision-SUPPORT only.          │
│       Final authority: qualified judge advocate.                         │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘

████████████████████████████████████████████████████████████████████████████
█                      █  END OF ARCHITECTURE DIAGRAM                       
████████████████████████████████████████████████████████████████████████████

########################################################################
#  PROJECT MEDUSA - DASHBOARD (Envision text mode)
#  Multi-Agent Legal Assistant for Military Justice (UCMJ / MCM)
########################################################################

========================================================================
| CASE SUMMARY
========================================================================
  Case ID............: CASE-2024-0042
  Correlation ID.....: corr-261c688086d2
  Coordination mode..: central_planner
  Query..............: Analyze Article 121 elements and assess motion-to-dismiss success.
  Pseudonymized text.:
    [RANK_NAME-CA7DA1] is accused under UCMJ Article 121 of wrongfully taking government property with the intent to permanently deprive the unit of equipment. [PERSON-CC704C] (witness) stated he saw the property removed. Contact: [EMAIL-F4A1C5], [PHONE-C78557], SSN [SSN-7ABC0E].

========================================================================
| LEGAL RESEARCH (GraphRAG: KG traversal + vector search)
========================================================================
  Rule  : Wrongful taking of property of another with intent to permanently deprive.
  Source: UCMJ Art. 121 (Larceny & Wrongful Appropriation)
  Chunk : Under UCMJ Article 121, larceny requires a wrongful taking and intent to permanently deprive the owner of property.
  Chunk : Wrongful appropriation differs from larceny by intent to temporarily deprive.
  Chunk : Article 134 covers general offenses prejudicial to good order and discipline.

========================================================================
| RISK PREDICTION (XGBoost-emulated + SHAP)
========================================================================
  P(motion success)..: [##################------------] 0.590
  Appeal likelihood..: [#######-----------------------] 0.236
  Predicted complexity: high
  SHAP attributions:
    doc_length           + 0.0138
    num_entities         - 0.2000
    mentions_intent      + 0.3000
    mentions_property    + 0.2500
  Fairness............: {'equalized_odds_monitored': True}
  Model card..........: {'model': 'XGBoost-emulated', 'version': '1.0.0'}

========================================================================
| RESOURCE ALLOCATION (constraint-optimization heuristic)
========================================================================
  Attorneys assigned.: 3
  Paralegals assigned: 2
  Priority score.....: 1.18
  Recommendation.....: proceed_to_trial

========================================================================
| SYLLOGISTIC DRAFT (Legal Syllogism Prompting via Claude)
========================================================================
  [Claude:claude-medusa-stub-v1#587eae48] MAJOR PREMISE (Rule): see retrieved UCMJ article. MINOR PREMISE (Facts): see case facts. CONCLUSION: application of rule to facts yields a reasoned judgment (decision-support only).
  
  [DISCLAIMER] This output is AI-generated decision-support and does NOT constitute legal advice. A qualified judge advocate must review and make all final decisions.

========================================================================
| DE-ANONYMIZATION (RBAC + ABAC + MFA -- Sec 4.5)
========================================================================
  Authorized CaseLead -> [SSN-7ABC0E] = 123-45-6789
  Unauthorized request -> BLOCKED (RBAC/ABAC denied: requires CaseLead on this CaseID + MFA.)

========================================================================
| RLHF FEEDBACK (Sec 5.2)
========================================================================
  Submitted ratings..: {'Legal Accuracy': 4, 'Citation Quality': 5, 'Clarity': 4, 'Completeness': 3}
  Computed reward....: [#########################-----] 0.840

========================================================================
| PROVENANCE AUDIT TRAIL (hash-chained ledger -- Sec 6.1)
========================================================================
  Ledger integrity valid: True
  12 events for this query:
    [1781199194654] Orchestrator                               route_query
    [1781199194654] EvidenceCollectionAgent                    ingest_document
    [1781199194654] EvidenceCollectionAgent@1.0.0              handle:evidence_collection
    [1781199194654] GovernanceDesignAgent@1.0.0                handle:governance_design
    [1781199194654] GovernanceCoordinationAgent@1.0.0          handle:governance_coordination
    [1781199194654] LegalResearchAgent@1.0.0                   handle:legal_research
    [1781199194654] LegalDraftingAgent                         llm_reasoning_trace
    [1781199194654] LegalDraftingAgent@1.0.0                   handle:legal_drafting
    [1781199194655] RiskPredictionAgent@1.0.0                  handle:risk_prediction
    [1781199194655] ResourceAllocationAgent@1.0.0              handle:resource_allocation
    [1781199194655] Orchestrator                               assemble_response
    [1781199194657] RLHF                                       ingest_feedback

########################################################################
#  END OF DASHBOARD
#  run `streamlit run project_medusa.py` for the Streamlit web UI
########################################################################
