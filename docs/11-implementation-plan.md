# 11 — Plano de Implementação

## 1. Estratégia

Implementar vertical slices testáveis. Cada etapa deve entregar capacidade utilizável e benchmarkável.

## Fase 0 — Foundation

### Entregáveis
- estrutura Python;
- pyproject;
- config tipada;
- logging estruturado;
- Docker Compose;
- PostgreSQL/pgvector;
- MinIO;
- Redis;
- migrations;
- CI.

### Critérios
- app sobe localmente;
- health checks;
- migrations idempotentes;
- baseline de unit/integration tests.

## Fase 1 — Case + Ingestion

### Módulos
- Case domain;
- SourceFile domain;
- upload;
- hash;
- manifest;
- object storage adapter;
- processing job.

### Testes
- arquivo duplicado;
- arquivo corrompido;
- upload interrompido;
- hash consistente;
- ACL.

## Fase 2 — Document Intelligence

### Módulos
- native extractor;
- OCR provider interface;
- page/block model;
- quality gate;
- normalized artifacts.

### Corpus de teste
- PDF nativo;
- scan bom;
- scan ruim;
- rotated page;
- tabela;
- documento híbrido.

## Fase 3 — Chunking + Index

### Módulos
- structural chunker;
- SourceSpan;
- embedding provider;
- pgvector;
- FTS.

### Benchmark inicial
Criar perguntas factuais e gold spans.

## Fase 4 — Hybrid Retrieval

### Módulos
- query understanding;
- lexical retriever;
- dense retriever;
- metadata retriever;
- RRF;
- reranker;
- context expansion;
- Evidence Pack v1.

### Gate
Comparar:
- lexical only;
- vector only;
- hybrid;
- hybrid + reranking.

Não avançar sem métricas.

## Fase 5 — Answer + Citation

### Módulos
- reasoning provider;
- response schema;
- citation formatter;
- source viewer locator;
- audit query run.

### Gate
Medir:
- grounded claim ratio;
- citation precision;
- unsupported claim rate.

## Fase 6 — Semantic Knowledge

### Módulos
- Entity;
- EntityMention;
- EntityAlias;
- Claim;
- Event;
- Evidence;
- extraction runs.

Começar persistindo em PostgreSQL.

### Gate
Dataset anotado e F1 mínimo definido empiricamente.

## Fase 7 — Entity Resolution + Timeline

### Módulos
- candidate generation;
- alias matching;
- ambiguity handling;
- human review;
- timeline query.

### Gate
Homônimos e aliases fazem parte da suíte.

## Fase 8 — Knowledge/Evidence Graph

### Módulos
- Neo4j adapter;
- graph projection;
- traversal API;
- graph retriever;
- Evidence Graph.

### Gate
Demonstrar ganho em perguntas multi-hop.

## Fase 9 — Specialized Reasoning

### Workflows
- claim comparison;
- contradiction;
- corroboration;
- timeline reconstruction;
- relationship analysis.

### Gate
Benchmark específico para cada workflow.

## Fase 10 — Verification

- citation validator;
- critic/verifier;
- re-retrieval controlado;
- abstention policy.

## Fase 11 — Production Hardening

- OIDC;
- RBAC/ABAC;
- HA;
- PITR;
- Temporal;
- dashboards;
- rate limits;
- cost control;
- security testing;
- disaster recovery.

## Fase 12 — Legal-grade Controls

- WORM;
- dual OCR;
- formal review;
- versioned evidence;
- locked benchmarks;
- reproducible export;
- forensic audit.

## 2. Épicos sugeridos

1. Foundation
2. Case Management
3. Secure Ingestion
4. Document Intelligence
5. Chunking & Provenance
6. Hybrid Search
7. Evidence Pack
8. Grounded Answering
9. Semantic Extraction
10. Entity Resolution
11. Timeline
12. Evidence Graph
13. Advanced Reasoning
14. Verification
15. Security & Compliance
16. Observability & Evaluation

## 3. Definition of Done por módulo

Todo módulo deve possuir:
- interface/contract;
- domain tests;
- unit tests;
- integration tests;
- structured logs;
- metrics;
- error model;
- docs;
- configuração externalizada;
- sem secrets/hardcode;
- type hints;
- migration quando houver persistência.

## 4. Ordem de prioridade

1. proveniência;
2. retrieval;
3. avaliação;
4. resposta;
5. semântica;
6. grafo;
7. agentes avançados.

Isso evita construir uma camada sofisticada de reasoning sobre retrieval frágil.
