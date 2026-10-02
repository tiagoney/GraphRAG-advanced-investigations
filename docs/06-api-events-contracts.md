# 06 — Contratos de API e Eventos

## 1. Convenções

API REST no MVP. Event bus para processamento assíncrono.

Padrões:
- UUIDs;
- ISO-8601 UTC;
- idempotency keys;
- correlation IDs;
- schema version;
- Problem Details para erros.

## 2. Endpoints principais

### Cases
- POST /v1/cases
- GET /v1/cases/{case_id}
- POST /v1/cases/{case_id}/members

### Files
- POST /v1/cases/{case_id}/files
- POST /v1/cases/{case_id}/files/{source_file_id}/complete
- GET /v1/cases/{case_id}/files/{source_file_id}/status

O início de upload retorna source_file_id, upload target e idempotency key.

### Documents
- GET /v1/cases/{case_id}/documents
- GET /v1/cases/{case_id}/documents/{document_id}
- GET /v1/cases/{case_id}/documents/{document_id}/pages/{page_number}

### Entities
- GET /v1/cases/{case_id}/entities
- GET /v1/cases/{case_id}/entities/{entity_id}
- POST /v1/cases/{case_id}/entities/{entity_id}/resolve

### Claims, Events e Evidence
- GET /v1/cases/{case_id}/claims
- GET /v1/cases/{case_id}/events
- GET /v1/cases/{case_id}/evidence

### Query
POST /v1/cases/{case_id}/queries

Entrada:
- query;
- mode;
- optional time range;
- optional entity filters;
- response detail level.

Saída:
- query_run_id;
- answer;
- citations;
- confidence/sufficiency;
- evidence_pack_id;
- diagnostics permitidos pelo papel do usuário.

### Review
- POST /v1/reviews
- POST /v1/reviews/{review_id}/decision

## 3. Citation contract

Cada citation:
- citation_id;
- source_file_id;
- document_id;
- page_number;
- source_span_id;
- exact_text;
- bbox optional;
- source_hash;
- viewer locator.

A resposta não deve depender de citation sem source_span_id.

## 4. Error model

Campos:
- type;
- title;
- status;
- code;
- detail;
- correlation_id;
- retryable;
- errors[].

Códigos iniciais:
- CASE_NOT_FOUND
- ACCESS_DENIED
- FILE_VALIDATION_FAILED
- OCR_FAILED
- PROCESSING_INCOMPLETE
- RETRIEVAL_FAILED
- MODEL_PROVIDER_UNAVAILABLE
- EVIDENCE_INSUFFICIENT
- CITATION_VALIDATION_FAILED

## 5. Eventos

Envelope:
- event_id;
- event_type;
- schema_version;
- occurred_at;
- producer;
- case_id;
- aggregate_id;
- correlation_id;
- causation_id;
- payload.

### Ingestão
- SourceFileReceived
- SourceFileValidated
- SourceFileStored
- DocumentCreated
- DocumentExtractionRequested
- DocumentExtracted
- DocumentQualityFlagged
- DocumentStructured
- ChunksCreated

### Semânticos
- EmbeddingsRequested
- EmbeddingsGenerated
- EntityExtractionCompleted
- ClaimsExtracted
- EventsExtracted
- EvidenceCandidatesExtracted
- EntityResolutionRequested
- GraphProjectionRequested
- GraphProjectionUpdated

### Consulta
- QueryReceived
- QueryPlanned
- RetrievalCompleted
- EvidencePackBuilt
- ReasoningCompleted
- CitationValidationCompleted
- QueryCompleted
- QueryAbstained

## 6. Idempotência

Commands mutáveis aceitam Idempotency-Key.
Consumers devem usar inbox/outbox ou mecanismo equivalente.
Não depender de entrega exactly-once do broker.

## 7. Event versioning

Nunca alterar semanticamente evento já publicado. Mudança incompatível exige novo schema version e compatibilidade de consumers.

## 8. API security

- OAuth2/OIDC
- JWT curto
- case-level authorization
- server-side enforcement
- signed URLs curtas
- rate limits
- audit logging

## 9. Contrato de viewer

O frontend deve abrir documento, página e highlight do span/bounding box citado, além de contexto adjacente.
