# 10 — Roadmap: MVP, Produção e Alta Confiabilidade

## 1. Princípio

A plataforma evolui por capacidades comprovadas por testes e benchmarks, não por quantidade de agentes ou serviços.

## 2. Nível 1 — MVP / Prova de Valor

### Objetivo

Provar:
- ingestão confiável;
- busca híbrida;
- citações;
- respostas grounded;
- viabilidade em casos grandes.

### Componentes

- FastAPI;
- PostgreSQL + pgvector;
- PostgreSQL FTS;
- MinIO/S3;
- Redis;
- worker;
- um provider de Document Intelligence;
- um embedding provider;
- um reranker;
- um LLM de reasoning;
- frontend simples.

### Funcionalidades

- case;
- upload;
- OCR;
- pages/blocks/chunks;
- hybrid retrieval;
- RRF;
- reranking;
- context expansion;
- Evidence Pack v1;
- answer + citations;
- audit básico.

### Sem Neo4j obrigatório

Entidades e relações podem ficar inicialmente em PostgreSQL.

### Gate de saída

- benchmark criado;
- Recall@K conhecido;
- citation precision medida;
- zero cross-case leak nos testes;
- respostas reproduzíveis em nível de evidência.

## 3. Nível 2 — Produção

### Objetivo

Operar múltiplos casos e usuários com robustez.

Adicionar:
- HA;
- OIDC;
- RBAC/ABAC;
- workflow engine;
- Neo4j;
- entity resolution;
- claims;
- events/timeline;
- Case Evidence Graph;
- specialized reasoning workflows;
- citation verifier;
- dashboards;
- CI/CD completo;
- backup/PITR;
- rate limits;
- cost controls.

### Gate de saída

- SLOs definidos;
- load test;
- disaster recovery test;
- evaluation regression pipeline;
- security assessment;
- audit trail completo.

## 4. Nível 3 — Alta Confiabilidade Jurídica

### Objetivo

Maximizar verificabilidade e reduzir risco de erro em análises críticas.

Adicionar:
- immutable/WORM raw storage;
- dual OCR para baixa confiança;
- human review formal;
- bitemporal/versioned semantic records;
- second-pass independent verification;
- políticas de abstention mais rígidas;
- locked gold datasets;
- provenance até bbox/hash;
- relatórios de evidência exportáveis;
- model/index snapshotting;
- forensic audit trail;
- policy-based approvals.

### Gate de saída

- evidências críticas podem ser reproduzidas;
- cada afirmação material está citada/verificada;
- divergências de OCR críticas são revisadas;
- release gate baseado em benchmark;
- auditoria independente possível.

## 5. Matriz

| Capacidade | MVP | Produção | Alta confiança |
|---|---|---|---|
| Object storage | S3/MinIO | managed + versioning | WORM/object lock |
| OCR | single provider | fallback | dual + human review |
| PostgreSQL | single/managed | HA + PITR | HA + temporal/audit |
| Vector | pgvector | pgvector ou dedicado se necessário | benchmark + snapshot |
| Lexical | PostgreSQL FTS | FTS/OpenSearch se necessário | regression tested |
| Graph | relacional | Neo4j | versioned projection |
| Claims/events | básico | completo | review/versioning |
| Reranker | sim | tuning | validated/ensemble opcional |
| Agents | mínimo | especializados | workflows rígidos |
| Citation verifier | parcial | sim | obrigatório |
| Human review | manual | por exceção | formal |
| Audit | básico | completo | forensic |
| Evaluation | starter set | CI regression | locked benchmark |

## 6. Critério para adicionar tecnologia

Adicionar complexidade somente quando:
1. requisito funcional exigir;
2. benchmark demonstrar ganho;
3. escala demonstrar gargalo;
4. auditoria ou governança exigir.

## 7. Evitar

- microservices cedo sem razão;
- vector DB separado por moda;
- GraphRAG sem modelo semântico;
- agentes como substituto de retrieval;
- confundir modelo maior com maior confiabilidade.
