# Mhd-Y Policy Evidence RAG Prototype

A no-code/low-code policy question-answering prototype built with n8n, Gemini Embeddings, Qdrant and an Ollama chat model. It has separate document-ingestion and question-answering workflows. All policy content in this repository is fictional test data.

> **Status:** Prototype; initial ingestion, duplicate detection and one payment-term Q&A scenario were reported as successful. Most planned tests and real form-upload/PDF paths remain unverified. Workflow exports in this repository are sanitised and inactive; credentials must be reconnected after import.

## Problem

Policy readers may spend time searching long handbooks for exact obligations, limits and exceptions. This prototype experiments with retrieving relevant passages and supplying them to an LLM as context for a cited answer.

## Architecture

### Ingestion workflow

Upload/load fictional policy document → validate type/content and chunk settings → compute content hash and check ingestion log → extract and split text → generate Gemini embeddings → store chunks and metadata in Qdrant collection `mhdy_policy_chunks` → log chunk count and status in n8n Data Table `policy_ingestion_log`.

Default chunk configuration: 800 characters with 120-character overlap.

### Question-answering workflow

Chat question → validate question → embed question with Gemini Embeddings → retrieve up to four similar chunks from Qdrant → build source-labelled evidence → generate an answer with the Ollama model → format the answer and append retrieved evidence for troubleshooting.

The answer prompt directs the model to use only retrieved passages, treat document contents as untrusted evidence, cite available source metadata, distinguish explicit text from interpretation, disclose insufficient evidence, and require human review. This design reduces risk but does not guarantee correctness; evaluation is still required.

## Tech stack

- **n8n:** visual workflow orchestration and JavaScript Code nodes
- **Gemini Embeddings:** maps document chunks and queries into vectors
- **Qdrant:** persistent vector storage and similarity retrieval
- **Ollama Chat Model (`gemma4:31b-cloud` in the export):** answer generation
- **n8n Data Table:** ingestion log and duplicate-detection status

## Evidence available so far

- Ingestion execution 33: the sample handbook indexed as two chunks and a corresponding `indexed` log row was reported.
- Chat execution 34: a question about the maximum supplier payment term retrieved two passages and answered with a 60-day standard term and Finance Director written-approval exception.
- Duplicate executions 35 and 36: the same document was skipped instead of being indexed again.
- Longer-handbook execution 39: the workflow execution succeeded; verify its exact indexed chunk count from the data table before making a count claim.

These are individual observed cases, not an accuracy benchmark. The exported chat workflow JSON's original `active` flag differed from the project notes; this repository deliberately sets both portable exports inactive. Verify status in n8n before relying on any live workflow.

## Repository map

- `workflow/ingestion-workflow.json` — sanitised, inactive ingestion workflow export
- `workflow/rag-chat-workflow.json` — sanitised, inactive query workflow export
- `sample-data/` — fictional 10-policy handbook extracted from the workflow's sample generator
- `docs/architecture.md` — node-by-node architecture
- `docs/setup-guide.md` — import and credential configuration notes
- `docs/interview-notes.md` — concepts and truthful project explanation
- `tests/test-cases.md` — planned tests
- `tests/test-results.md` — observed results and outstanding tests
- `screenshots/README.md` — list of real screenshots to add

## Setup overview

1. Import the JSON workflow files into n8n. They are intentionally inactive and have no embedded credential references.
2. Reconnect Qdrant and Google Gemini credentials to the matching nodes. Configure the Ollama endpoint and credential for the chat model.
3. Ensure both workflows use the same embedding model (`models/gemini-embedding-001`) and Qdrant collection (`mhdy_policy_chunks`). Changing embedding models requires re-embedding/re-indexing the documents.
4. Ensure the n8n Data Table `policy_ingestion_log` exists with the fields expected by the workflow.
5. Ingest the fictional handbook, check the log, then ask test questions in the chat UI.
6. Record actual outcomes in `tests/test-results.md`.

## Important limitations

- Most of the planned evaluation suite is not yet run.
- The real form-upload path and PDF ingestion are unverified.
- Accuracy and retrieval precision have not been measured.
- Source/page metadata may be unavailable for Markdown documents.
- The chatbot's appended retrieval report is evidence for debugging, not proof that every generated statement is supported.
- This is a fictional learning prototype, not a production compliance system or legal advice.

## Development note

The n8n AI assistant helped generate the initial workflows. The project involved configuring and reviewing workflow logic, testing initial ingestion and retrieval cases, and troubleshooting embedding credentials and ingestion-status behaviour. It is not represented as entirely hand-coded from scratch.

## Author

Mohamed Yousuf
