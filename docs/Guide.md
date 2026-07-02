# 🏛️ Athena System Development Roadmap

This document details the development lifecycle of Athena from **Day 1** to **Version 1.0 (MVP)**. Every development phase concludes with a completely demonstrable, usable milestone.

---

## 📈 The Big Picture Flow

```text
Phase 0: Blueprint
   │
   ▼
Phase 1: Working Web Application (Skeleton)
   │
   ▼
Phase 2: RAG Pipeline (Syllabus Knowledge Base)
   │
   ▼
Phase 3: Multi-Agent Workflows (Specialists Layer)
   │
   ▼
Phase 4: Exam Generator Engine (Database & PDF Rendering)
   │
   ▼
Phase 5: Intelligent Tutor (Adaptive Learning Loop)
   │
   ▼
Phase 6: Production Refinement (Testing & Launch Prep)
```

---

## 🚀 Phase 0: Planning & System Design
**Duration:** 2–3 Days

### Objective
Finalize system requirements, architecture boundaries, and structural baselines before executing code. Think of this as designing a house blueprint before laying the foundation.

### Core Core Execution
* **Finalize Requirements:** Document absolute user capabilities.
  * *Example Capabilities:* Ask doubts, generate topical papers, authenticating user sessions, downloading generated PDFs, browsing historical logs, isolating subjects, and adjusting difficulty tiers.
* **Isolate Tech Stack:** Formally commit to all stack layers.
  * **Frontend:** React, Tailwind CSS
  * **Backend:** FastAPI
  * **AI/Orchestration:** OpenAI APIs, LangGraph, ChromaDB
  * **Database:** PostgreSQL
* **Design System Architecture:** Map multi-agent processing layers down to the core databases.
  ```text
  Frontend ──> Backend ──> Manager Agent ──> Subject Agents ──> Vector/Relational DBs
  ```
* **Construct Data Models:** Structurally map database relational schemas.
  * *Question Table Schema:* `QuestionID`, `Subject`, `Topic`, `Difficulty`, `Marks`, `Year`, `Paper`
  * *Student Table Schema:* `ID`, `Username`, `Email`, `Password`
* **Draft API Specifications:** 
  * `POST /chat`
  * `POST /login`
  * `POST /generate-paper`
  * `GET /topics`
  * `GET /history`
* **Repository Strategy:** Standardise branching models: `main`, `develop`, `feature/frontend`, `feature/backend`.

### Deliverables
- [ ] Initialize GitHub Repository & structural folders
- [ ] Documented relational database schema designs
- [ ] OpenAPI-compliant endpoint design specifications
- [ ] End-to-end system architecture design diagrams

---

## 🚀 Phase 1: Foundation
**Duration:** 1 Week

### Objective
Create an end-to-end functional application skeleton. Complete the core client-server communication channels using mock inputs/outputs. **No AI implementations occur here.**

### Core Execution
* **Frontend Scaffolding:** Build layout frames for the Home View, Login UI, Main Dashboard, Chat Workspaces, and the Paper Generator dashboard.
* **Backend Bootstrapping:** Spin up a raw FastAPI server instance serving standard test endpoints (`GET /`, `POST /chat`). Return static string mock payloads.
* **Authentication Pipelines:** Embed stateless JSON Web Token (JWT) or session workflows handling User Registration, User Login, and Session Termination.
* **End-to-End Handshakes:** Validate client-to-server networks (Frontend sends `"Hello"` $\rightarrow$ Backend logs request $\rightarrow$ Returns `"Hi"`).

### Deliverables
- [ ] Fully browseable frontend interfaces running locally
- [ ] Functional, secure registration and login pipelines
- [ ] Responsive FastAPI server serving operational endpoints
- [ ] Validated client-server communication network loops

---

## 🚀 Phase 2: RAG System
**Duration:** 2 Weeks

### Objective
Ground the system's foundational knowledge boundaries using custom learning materials. Train the model to respond strictly using verified academic inputs.

### Core Execution
* **Data Gathering:** Standardize input document sources (Notes, Past Examination Papers, Definitive Mark Schemes, Examiner Assessment Reports).
* **Text Extraction Pipelines:** Write processing scripts transforming arbitrary layout PDFs into normalized raw structural plain text string payloads.
* **Chunking Optimization:** Split target text segments down into semantic chunks sized between $200\text{--}500$ words to retain dense context boundaries.
* **Vector Embeddings Store:** Pass text segments through target embedding models and persist generated vectors directly inside localized ChromaDB collections.
* **Build Retriever Interface:** Build search scripts that query the vector store for top-$K$ highly relevant context blocks based on incoming user queries.
* **Strict Prompt Engineering:** Build LLM system prompts enforcing hard constraints: *Use ONLY the provided context blocks. Do not synthesize outside facts. Flag unknown context clearly.*

### Deliverables
- [ ] Scalable data ingestion engine scripts
- [ ] Populated ChromaDB instances holding localized syllabus vectors
- [ ] Verified semantic search engines returning contextually matched chunks
- [ ] Grounded RAG chat loops providing citation-backed answers with zero external hallucinations

---

## 🚀 Phase 3: Multi-Agent Workflow
**Duration:** 1 Week

### Objective
Transition away from monolithic LLM processing loops by distributing complex reasoning workflows to a team of specialized agents coordinated via LangGraph.

### Core Execution
* **Intent Parser Agent:** Extract target data variables (Action Task, Subject, Core Topic Focus, Difficulty Parameters) from incoming raw text string commands.
* **Manager Orchestrator Agent:** Track global loop state parameters and dynamically route process handling to specific downstream expert nodes.
* **Domain Subject Agents:** Establish individual node personalities configured with custom system prompts for Physics, Chemistry, and Biology experts.
* **Syllabus Verification Agent:** Program a specialized supervisor node tasked with reviewing generated answers to filter out extra-syllabus concepts before compilation.
* **Output Formatter Node:** Clean structural JSON and raw model logs into clean, readableMarkdown outputs for client presentation.

### Deliverables
- [ ] Fully mapped LangGraph state graph charts
- [ ] Intent engine capable of routing varied complex student requests
- [ ] Specialized domain node blocks executing tasks under isolation
- [ ] Post-generation verification workflows ensuring safe outputs

---

## 🚀 Phase 4: Question Generator
**Duration:** 2–3 Weeks

### Objective
Build the underlying core processing engine responsible for indexing complex examination layouts, tracking granular evaluation criteria, and rendering exam papers.

### Core Execution
* **Structural Paper Parsing:** Break complex source exam papers down into specific granular relational entities containing sub-questions, mark weightings, tables, and target correct answers.
* **Metadata Taxonomy Processing:** Run text structures through classification nodes to index them against attributes like Difficulty, Main Topic, and Technical Calculation requirements.
* **Structured Data Persistence:** Save questions securely inside relational PostgreSQL structures to enable lightning-fast complex filter querying.
* **Algorithmic Selection Logic:** Program optimization algorithms that fetch questions perfectly satisfying complex filter configurations (e.g., Target 40 Marks total $\rightarrow$ fetches 6, 8, 4, 10, and 12-mark questions mapping to specific topics).
* **PDF Rendering Framework:** Write server-side templates capable of outputting structural examination layouts matching standardized testing house designs.

### Deliverables
- [ ] Automated question indexing and database ingestion pipelines
- [ ] Granularly searchable relational database filled with structural exam problems
- [ ] Optimization algorithms matching selection criteria precisely
- [ ] Server-side layout render engine outputting print-ready exam PDFs

---

## 🚀 Phase 5: Intelligent Tutor
**Duration:** 1–2 Weeks

### Objective
Evolve system chat components from basic informational question-and-answer frames into active, adaptive, and pedagogical diagnostic tutoring dialogues.

### Core Execution
* **Socratic Instructional Loop:** Adjust model behavior from direct answer resolution to interactive educational prompts. Require the model to query user understanding step by step.
* **Dynamic Student Profiling:** Track real-time metric updates monitoring a user's Weak Subjects, Proven Strengths, Comprehension Confidence, and repetitive Core Mistakes.
* **Personalized Content Routing:** Build recommendation hooks that trigger automatically when performance dips below thresholds in a certain topic, serving targeted remediation assets.

### Deliverables
- [ ] Conversational tutoring states running active educational dialogues
- [ ] Persistent state tables capturing dynamic student performance profiles
- [ ] Automated recommendation logic providing personal learning routes

---

## 🚀 Phase 6: Testing & Refinement
**Duration:** 2 Weeks

### Objective
Transform an unpolished collection of functional modules into a production-grade, highly optimized Minimum Viable Product (MVP).

### Core Execution
* **Hallucination Evaluation:** Execute batch automation frameworks passing 100+ evaluation questions through the pipeline to track Accuracy, Validity, and Syllabus drift.
* **Retrieval Metric Tuning:** Audit vector retrieval precision and optimize chunk sizing metrics or overlap parameters to ensure top-tier document context matches.
* **UI/UX Polish:** Apply smooth CSS transitions, loading state spinners, full color-mode styling, and chat interface optimizations.
* **Performance Engineering:** Implement caching layers, concurrent execution loops, and token footprint management strategies to drop round-trip latency.
* **Comprehensive Documentation:** Finalize end-to-end installation documentation, database architecture manifests, and detailed API references.

### Deliverables
