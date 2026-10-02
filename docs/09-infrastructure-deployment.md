# 09 — Infraestrutura e Deployment

## 1. Ambientes

- local;
- development;
- staging;
- production.

Dados reais confidenciais não devem entrar em development sem política específica.

## 2. Local/MVP

Docker Compose:
- api;
- worker;
- postgres + pgvector;
- redis;
- minio;
- optional local provider adapters.

Objetivo: desenvolvimento reproduzível.

## 3. Produção

Componentes lógicos:
- load balancer/API gateway;
- frontend;
- API service;
- worker pools;
- workflow engine;
- PostgreSQL HA;
- Redis;
- object storage;
- graph database;
- observability stack;
- secrets/KMS;
- identity provider.

## 4. Worker pools

Separar workloads:
- extraction;
- OCR;
- embedding;
- semantic extraction;
- graph projection;
- query async tasks.

Razões:
- perfis de CPU/memória/GPU diferentes;
- scaling independente;
- isolamento de falhas.

## 5. Workflow engine

MVP pode usar queue + task worker.

Produção deve avaliar Temporal ou equivalente para:
- durable workflows;
- retries;
- timers;
- state;
- compensação;
- visibility.

## 6. PostgreSQL

MVP:
- single instance ou managed;
- backup;
- migrations.

Produção:
- managed HA preferencialmente;
- PITR;
- read replicas conforme necessidade;
- connection pool;
- pgvector tuning.

## 7. Graph DB

Introduzir quando GraphRAG entrar no roadmap.

Requisitos:
- backup;
- reconstrução via projection;
- case_id em nós/relações;
- índices de entidade;
- versionamento da projeção.

## 8. Object storage

Produção:
- lifecycle;
- versioning;
- encryption;
- replication conforme RTO/RPO;
- object lock quando exigido.

## 9. Networking

- private subnets para data stores;
- egress control;
- provider allowlist quando viável;
- TLS interno/externo;
- WAF/rate limiting no edge.

## 10. Secrets

Nunca armazenar em:
- repositório;
- .env versionado;
- logs;
- prompts.

Usar Secret Manager/Vault/KMS.

## 11. CI/CD

Pipeline mínimo:
1. lint;
2. type check;
3. unit tests;
4. integration tests;
5. migration validation;
6. security scan;
7. build image;
8. SBOM;
9. deploy staging;
10. smoke/E2E;
11. approval/deploy production.

## 12. Infrastructure as Code

Terraform/OpenTofu ou equivalente.

Recursos devem ser reproduzíveis e revisáveis via PR.

## 13. Backup e recovery

Definir RPO/RTO por ambiente.

Testar restore periodicamente.

Backups sem teste de restauração não atendem requisito de resiliência.

## 14. Escala

Escalar horizontalmente:
- API;
- stateless retrieval services;
- workers.

Escala de storage/index orientada por métricas reais.

Evitar vector DB dedicado antes de pgvector demonstrar limite.

## 15. Custo

Medir custo por:
- página processada;
- embedding;
- query;
- reasoning;
- provider;
- case.

Políticas possíveis:
- modelo econômico primeiro;
- modelo mais caro apenas para verificação/raciocínio complexo;
- cache de embeddings;
- evitar reprocessamento via idempotência.

## 16. Disaster recovery

Alta confiança:
- runbook;
- restore drill;
- corruption detection;
- secondary region quando justificável;
- export de manifests/hashes.
