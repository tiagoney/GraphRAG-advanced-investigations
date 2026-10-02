# 02 — Modelo de Domínio e Persistência

## 1. Agregados principais

### Case

Raiz de isolamento de segurança e investigação.

Campos mínimos:
- id UUID;
- external_reference;
- title;
- description;
- status;
- confidentiality_level;
- retention_policy_id;
- created_at;
- closed_at.

### SourceFile

Representa o arquivo recebido.

Campos:
- id;
- case_id;
- parent_source_file_id;
- original_filename;
- media_type;
- byte_size;
- sha256;
- storage_uri;
- ingestion_source;
- ingested_at;
- immutable_version_id.

### Document

Representa uma unidade documental lógica.

Um SourceFile pode gerar múltiplos Documents, por exemplo um pacote ou e-mail com anexos.

Campos:
- id;
- case_id;
- source_file_id;
- document_type;
- title;
- author;
- document_date;
- language;
- page_count;
- processing_status.

### Page

- id;
- document_id;
- page_number;
- width;
- height;
- native_text_available;
- ocr_confidence;
- page_image_uri.

### DocumentBlock

Tipos:
- paragraph;
- heading;
- table;
- figure;
- header;
- footer;
- footnote;
- list;
- signature;
- form_field.

Campos:
- id;
- page_id;
- block_type;
- reading_order;
- text;
- bbox;
- confidence;
- parent_block_id.

### Chunk

Unidade de retrieval.

Campos:
- id;
- case_id;
- document_id;
- section_id;
- page_start;
- page_end;
- text;
- token_count;
- chunk_strategy;
- content_hash;
- parent_chunk_id;
- previous_chunk_id;
- next_chunk_id.

### SourceSpan

Unidade mínima de proveniência.

Campos:
- id;
- block_id;
- page_id;
- start_offset;
- end_offset;
- bbox;
- exact_text;
- text_hash.

## 2. Entidades semânticas

### Entity

Campos:
- id;
- case_id;
- entity_type;
- canonical_name;
- normalized_name;
- resolution_status;
- confidence.

Tipos iniciais:
- PERSON;
- ORGANIZATION;
- LOCATION;
- ASSET;
- CONTRACT;
- ACCOUNT;
- COMMUNICATION;
- LEGAL_DOCUMENT;
- IDENTIFIER.

### EntityAlias

- id;
- entity_id;
- alias;
- normalized_alias;
- source_span_id;
- confidence.

### EntityMention

- id;
- entity_id nullable;
- candidate_entity_id nullable;
- source_span_id;
- mention_text;
- mention_type;
- extraction_run_id;
- confidence.

Nunca descartar menções não resolvidas.

## 3. Claims

Claim é uma proposição atribuível, não um fato automaticamente verdadeiro.

Campos:
- id;
- case_id;
- speaker_entity_id;
- subject_entity_id nullable;
- predicate;
- object_text;
- object_entity_id nullable;
- polarity;
- modality;
- asserted_at;
- event_time_start;
- event_time_end;
- source_span_id;
- extraction_run_id;
- confidence;
- review_status.

### ClaimRelation

Tipos:
- SUPPORTS;
- CONTRADICTS;
- QUALIFIES;
- REPEATS;
- SUPERSEDES;
- RELATED_TO.

Campos:
- source_claim_id;
- target_claim_id;
- relation_type;
- evidence_id nullable;
- confidence;
- detection_run_id;
- review_status.

## 4. Eventos

### Event

Campos:
- id;
- case_id;
- event_type;
- title;
- description;
- temporal_precision;
- time_start;
- time_end;
- location_entity_id;
- confidence;
- review_status.

### EventParticipant

- event_id;
- entity_id;
- role;
- source_span_id;
- confidence.

### EventEvidence

- event_id;
- evidence_id;
- relation_type.

## 5. Evidence

Evidence não significa "prova conclusiva". É um objeto que pode sustentar ou enfraquecer uma proposição.

Campos:
- id;
- case_id;
- evidence_type;
- title;
- description;
- primary_source_span_id;
- reliability_status;
- extraction_method;
- confidence;
- review_status.

Tipos:
- DOCUMENTARY;
- TESTIMONIAL;
- DIGITAL;
- EXPERT_REPORT;
- IMAGE;
- AUDIO;
- VIDEO;
- TABULAR;
- DERIVED.

### EvidenceRelation

- evidence_id;
- target_type;
- target_id;
- relation_type;
- strength;
- confidence;
- source_span_id.

## 6. Execuções/versionamento

Tabelas obrigatórias:
- processing_run;
- ocr_run;
- extraction_run;
- embedding_run;
- indexing_run;
- graph_projection_run;
- query_run;
- reasoning_run;
- verification_run.

Campos comuns:
- id;
- processor_name;
- processor_version;
- model_name;
- model_version;
- config_json;
- config_hash;
- started_at;
- finished_at;
- status;
- error_code;
- correlation_id.

## 7. Query e Evidence Pack

### QueryRun

- id;
- case_id;
- user_id;
- original_query;
- normalized_query;
- query_type;
- planner_version;
- authorization_snapshot_id;
- started_at;
- completed_at.

### RetrievalHit

Registrar, por estratégia:
- query_run_id;
- source_type;
- source_id;
- retriever;
- raw_score;
- normalized_rank;
- fused_rank;
- rerank_score;
- selected_for_evidence_pack.

### EvidencePack

- id;
- query_run_id;
- schema_version;
- build_config;
- pack_hash;
- created_at.

### EvidencePackItem

- evidence_pack_id;
- item_type;
- source_id;
- source_span_id;
- role: SUPPORTING | CONTRADICTING | CONTEXT | UNKNOWN;
- rank;
- confidence.

## 8. PostgreSQL extensions

MVP:
- pgvector;
- pg_trgm;
- uuid generation;
- full-text search.

Opcional:
- temporal/history extension ou tabelas de history explícitas.

## 9. Índices mínimos

- `source_file(case_id, sha256)` unique lógico;
- `document(case_id, document_date)`;
- `page(document_id, page_number)`;
- `chunk(case_id, document_id)`;
- GIN para full-text de chunk;
- HNSW para embedding;
- `entity(case_id, normalized_name)`;
- `claim(case_id, speaker_entity_id, asserted_at)`;
- `event(case_id, time_start, time_end)`;
- `source_span(page_id, start_offset, end_offset)`.

## 10. Regras de integridade

1. Todo chunk deve apontar para ao menos um SourceSpan.
2. Todo claim deve possuir fonte.
3. Toda relação de contradição automática deve guardar detector e versão.
4. Graph DB não pode ser fonte exclusiva de informação.
5. Exclusão lógica deve respeitar retenção e legal hold.
6. Correção humana não apaga output anterior: cria nova versão/revisão.
