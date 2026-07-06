# Project Athena — API Endpoints Documentation

**Document type:** Backend API Reference
**Location in repo:** `docs/api/API_ENDPOINTS.md`
**Audience:** Backend developers, frontend/client integrators, future JARVIS integration team
**Status:** Draft — Phase 1–4 scope

---

## 1. Purpose of This Document

This document defines the complete set of API endpoints exposed by the Athena backend (FastAPI). It is the single source of truth for how any client — the web app, a future desktop/mobile app, a voice assistant, or JARVIS itself — communicates with Athena.

Since the frontend is only one client of the backend, every endpoint here is designed to be **client-agnostic**: no assumptions about React, browsers, or session-based UI state.

Conventions used throughout:

- Base URL: `https://api.athena.ai/v1` (placeholder — update per environment)
- Format: JSON request/response bodies unless noted (e.g. PDF binary)
- Auth: Bearer token (`Authorization: Bearer <token>`) unless marked "Public"
- All timestamps: ISO 8601 UTC
- All IDs: UUID v4 unless noted

---

## 2. Endpoint Groups Overview

| Group | Prefix | Responsibility |
|---|---|---|
| Auth | `/auth` | Login, signup, token refresh, session |
| Chat / Doubt Clarification | `/chat` | Natural-language Q&A workflow |
| Intent | `/intent` | Standalone intent-parsing (internal/debug) |
| Question Generator | `/papers` | Custom question paper generation |
| Knowledge Base | `/knowledge` | Structured educational content CRUD |
| Question Bank | `/questions` | Past-paper question metadata CRUD |
| Retrieval | `/retrieve` | Direct vector-search access (internal/debug) |
| Ingestion | `/ingest` | Pipeline for adding new source material |
| User & Progress | `/users` | Profile, history, weak-topic tracking |
| Subjects | `/subjects` | Subject/topic taxonomy management |
| System | `/system` | Health, version, status |

---

## 3. Auth Endpoints — `/auth`

### `POST /auth/signup`
Create a new user account.
**Body:** `{ "email": string, "password": string, "name": string }`
**Response:** `201 { "user_id": uuid, "token": string }`

### `POST /auth/login`
**Body:** `{ "email": string, "password": string }`
**Response:** `200 { "token": string, "expires_at": datetime }`

### `POST /auth/refresh`
Refresh an expiring token.
**Body:** `{ "refresh_token": string }`
**Response:** `200 { "token": string }`

### `POST /auth/logout`
Invalidate current session token.
**Response:** `204`

### `GET /auth/me`
Return the authenticated user's profile.
**Response:** `200 { "user_id": uuid, "email": string, "name": string, "created_at": datetime }`

> Auth provider (Clerk/Firebase) may handle signup/login directly; in that case these endpoints act as thin verification/session-sync wrappers rather than owning credentials.

---

## 4. Chat / Doubt Clarification — `/chat`

This is the primary Feature 1 workflow: **User → Frontend → Intent Parser → Manager → Subject Agent → Retriever → Verifier → Formatter → User**.

### `POST /chat/message`
Submit a natural-language question.
**Body:**
```json
{
  "session_id": "uuid | null",
  "message": "Explain electrode potential",
  "mode": "teaching | revision | quick_answer | exam"
}
```
**Response:**
```json
{
  "session_id": "uuid",
  "intent": "explain",
  "subject": "chemistry",
  "topic": "electrochemistry",
  "answer": "string",
  "sources": [
    { "type": "syllabus | mark_scheme | past_paper", "reference": "string" }
  ],
  "verified": true
}
```
Internally this endpoint triggers the full multi-agent pipeline (Intent Parser → Manager → Subject Agent → Retriever → Verifier → Formatter) but exposes it as a single call to the client.

### `GET /chat/sessions`
List a user's past chat sessions.
**Response:** `200 [ { "session_id": uuid, "title": string, "updated_at": datetime } ]`

### `GET /chat/sessions/{session_id}`
Retrieve full message history for a session.
**Response:** `200 { "session_id": uuid, "messages": [ ... ] }`

### `DELETE /chat/sessions/{session_id}`
Delete a chat session.
**Response:** `204`

### `POST /chat/feedback`
Record whether a response was helpful/correct (feeds evaluation + verifier improvement).
**Body:** `{ "message_id": uuid, "rating": "correct | incorrect | unclear", "comment": "string | null" }`
**Response:** `200`

---

## 5. Intent Parser — `/intent` (internal / debug)

Exposed separately so the Intent Parser can be tested or reused outside the chat flow (e.g. by JARVIS's own router later).

### `POST /intent/parse`
**Body:** `{ "message": "string" }`
**Response:**
```json
{
  "intent": "explain | define | example | generate_paper | mark_answer | unknown",
  "subject": "string | null",
  "topic": "string | null",
  "requested_output": "text | pdf | flashcards | null",
  "confidence": 0.0
}
```

---

## 6. Question Paper Generator — `/papers`

Feature 2: users request a custom exam paper built only from real past-paper questions.

### `POST /papers/generate`
**Body:**
```json
{
  "subject": "chemistry",
  "topic": "electrochemistry",
  "difficulty": "easy | medium | hard | mixed",
  "total_marks": 40
}
```
**Response:**
```json
{
  "paper_id": "uuid",
  "status": "processing | ready",
  "download_url": "string | null"
}
```
Generation is deterministic (metadata search + selection), so this may respond synchronously for small papers or asynchronously (poll `/papers/{paper_id}`) for larger ones.

### `GET /papers/{paper_id}`
Check generation status / retrieve result.
**Response:** `200 { "paper_id": uuid, "status": "ready", "download_url": "string", "question_ids": ["uuid"] }`

### `GET /papers/{paper_id}/download`
**Response:** `200` (binary, `application/pdf`)

### `GET /papers/{paper_id}/mark_scheme`
Returns the corresponding mark scheme PDF/JSON for a generated paper.
**Response:** `200` (binary or JSON)

### `GET /papers`
List a user's previously generated papers.
**Response:** `200 [ { "paper_id": uuid, "subject": string, "topic": string, "created_at": datetime } ]`

---

## 7. Knowledge Base — `/knowledge`

CRUD over the **structured educational database** (definitions, theory, examples, formulae, common mistakes, examiner reports) — not raw PDFs.

### `GET /knowledge/{subject}/{topic}`
Retrieve the full structured knowledge object for a topic.
**Response:**
```json
{
  "subject": "chemistry",
  "topic": "electrochemistry",
  "definitions": ["..."],
  "theory": "string",
  "examples": ["..."],
  "formulae": ["..."],
  "common_mistakes": ["..."],
  "examiner_reports": ["..."]
}
```

### `POST /knowledge/{subject}/{topic}` *(admin)*
Create/update a topic's structured content.
**Body:** same shape as above.
**Response:** `200 / 201`

### `DELETE /knowledge/{subject}/{topic}` *(admin)*
**Response:** `204`

### `GET /knowledge/{subject}`
List all topics under a subject with summary metadata.
**Response:** `200 [ { "topic": string, "subtopics": [string], "last_updated": datetime } ]`

---

## 8. Question Bank — `/questions`

CRUD and search over individual past-paper questions and their metadata (Question ID, Subject, Topic, Subtopic, Difficulty, Marks, Question Type, Command Word, Paper, Year, Session, Variant, Diagram, Calculation, Mark Scheme).

### `GET /questions/search`
**Query params:** `subject, topic, subtopic, difficulty, marks_min, marks_max, year, command_word`
**Response:** `200 [ { question metadata objects } ]`

### `GET /questions/{question_id}`
**Response:** `200 { full question object including mark scheme reference }`

### `POST /questions` *(admin)*
Add a new question with full metadata.
**Body:** question metadata schema (see Question Database spec in `docs/database/`).
**Response:** `201 { "question_id": uuid }`

### `PUT /questions/{question_id}` *(admin)*
Update metadata for an existing question.
**Response:** `200`

### `DELETE /questions/{question_id}` *(admin)*
**Response:** `204`

---

## 9. Retrieval — `/retrieve` (internal / debug)

Direct access to the vector database retriever, useful for evaluation and debugging outside the full chat pipeline.

### `POST /retrieve/search`
**Body:** `{ "topic": "string", "keywords": ["string"], "context": "string | null", "top_k": 5 }`
**Response:** `200 [ { "chunk_id": uuid, "content": "string", "score": 0.0, "source": "string" } ]`

---

## 10. Ingestion Pipeline — `/ingest` (admin)

Follows the data flow: Source → Extract → Clean → Chunk → Embed → Store Metadata → Vector DB.

### `POST /ingest/document`
Upload a new source document (PDF, notes, syllabus, etc.) to be processed through the pipeline.
**Body:** multipart file + `{ "subject": string, "topic": string | null, "source_type": "syllabus | past_paper | mark_scheme | notes" }`
**Response:** `202 { "ingestion_id": uuid, "status": "queued" }`

### `GET /ingest/{ingestion_id}`
Check pipeline progress for a submitted document.
**Response:** `200 { "status": "extracting | chunking | embedding | storing | complete | failed", "error": "string | null" }`

### `GET /ingest`
List recent ingestion jobs.
**Response:** `200 [ { "ingestion_id": uuid, "status": string, "created_at": datetime } ]`

---

## 11. Users & Progress — `/users` (Phase 5+, scaffolded early)

### `GET /users/{user_id}/progress`
**Response:** `200 { "subjects": [ { "subject": string, "topics_covered": [string], "weak_topics": [string], "mastery_score": 0.0 } ] }`

### `POST /users/{user_id}/progress/event`
Log a learning event (question answered, paper attempted, chat completed).
**Body:** `{ "type": "chat | paper_attempt | flashcard", "subject": string, "topic": string, "result": "correct | incorrect | n/a" }`
**Response:** `201`

### `GET /users/{user_id}/flashcards`
**Response:** `200 [ { "flashcard_id": uuid, "topic": string, "due_at": datetime } ]` (spaced repetition, long-term feature)

---

## 12. Subjects & Taxonomy — `/subjects`

### `GET /subjects`
List all supported subjects.
**Response:** `200 [ { "subject": string, "topics": [string] } ]`

### `GET /subjects/{subject}/topics`
**Response:** `200 [ { "topic": string, "subtopics": [string] } ]`

### `POST /subjects` *(admin)*
Register a new subject (supports the domain-agnostic long-term vision — e.g. adding a university course or programming language later).
**Body:** `{ "subject": string, "description": string }`
**Response:** `201`

---

## 13. System — `/system`

### `GET /system/health`
**Public.** Liveness check.
**Response:** `200 { "status": "ok" }`

### `GET /system/version`
**Response:** `200 { "version": "string", "build": "string" }`

### `GET /system/status`
Detailed status of subsystems (vector DB, LLM provider, PDF generator) — useful for the JARVIS orchestrator when routing to Athena.
**Response:**
```json
{
  "vector_db": "ok | degraded | down",
  "llm_provider": "ok | degraded | down",
  "pdf_generator": "ok | degraded | down"
}
```

---

## 14. Error Response Format (applies to all endpoints)

```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": "object | null"
  }
}
```

Common codes: `unauthorized`, `not_found`, `validation_error`, `subject_unsupported`, `generation_failed`, `retrieval_empty`, `internal_error`.

---

## 15. Endpoint-to-Architecture Mapping (traceability)

| Endpoint | Architecture Component(s) Invoked |
|---|---|
| `POST /chat/message` | Intent Parser → Manager Agent → Subject Agent → Retriever → Verifier → Formatter |
| `POST /papers/generate` | Manager Agent → Question Generator → Question Database |
| `GET /knowledge/*` | Educational Database (direct, deterministic) |
| `GET /questions/*` | Question Database (direct, deterministic) |
| `POST /retrieve/search` | Retriever → Vector Database |
| `POST /ingest/document` | Full ingestion pipeline (Extract → Clean → Chunk → Embed → Store) |

This mapping is included so future contributors can trace any endpoint back to the multi-agent architecture described in the main project document, and so JARVIS-level integration work can identify which calls are safe to expose externally versus internal-only.

---

## 16. Notes for Future JARVIS Integration

- All endpoints above are namespaced under `/v1` to allow a future `/v2` without breaking JARVIS or other clients.
- Athena should expose a single "capability manifest" endpoint (e.g. `GET /system/capabilities`) once JARVIS integration begins, so the orchestrator can discover what Athena can do without hardcoding routes.
- Voice assistant and mobile clients should be able to use `/chat/message` and `/papers/generate` as their only two required integration points for MVP parity with the web app.

---

*This document should be updated alongside `docs/database/` (schema definitions) and `docs/architecture/` (agent responsibilities) as Athena moves through Phases 1–4.*
