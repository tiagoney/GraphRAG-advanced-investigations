# 07 — Segurança, Auditoria e Reprodutibilidade

## 1. Ativos críticos

- documentos originais;
- dados pessoais;
- informações sigilosas;
- evidências;
- prompts e respostas;
- índices;
- logs.

## 2. Riscos principais

- acesso cross-case;
- vazamento via retrieval;
- URL reutilizável de documento;
- prompt injection em documento;
- poisoning de índice;
- adulteração de fonte;
- resposta sem proveniência;
- ação administrativa não auditada.

## 3. Autorização

Princípio: authorization before retrieval.

case_id e ACL devem restringir:
- SQL;
- vector search;
- lexical search;
- graph traversal;
- object storage.

É proibido buscar globalmente e filtrar apenas ao final.

## 4. RBAC + ABAC

RBAC:
- admin;
- investigator;
- reviewer;
- reader;
- auditor.

ABAC:
- case membership;
- confidentiality;
- document classification;
- legal hold;
- atributos de tenant/projeto.

## 5. PostgreSQL

Produção:
- Row Level Security quando aplicável;
- roles separadas;
- usuário de migration distinto;
- application user sem DDL;
- secrets rotacionados;
- TLS;
- encryption at rest.

## 6. Object storage

- private buckets;
- server-side encryption;
- KMS;
- versioning;
- object lock/WORM em alta confiabilidade;
- signed URL curta;
- hash manifest.

## 7. Prompt injection documental

Texto do documento é dado não confiável.

Regras:
- instruções dentro de documentos nunca viram system/tool instructions;
- tool calls são definidos pelo workflow;
- evidence e instructions são delimitados;
- conteúdo suspeito pode ser sinalizado;
- ferramentas usam least privilege.

## 8. Audit log

Registrar:
- login;
- acesso ao caso;
- visualização/download;
- upload;
- alteração de ACL;
- revisão humana;
- consulta;
- exportação;
- reprocessamento;
- exclusão;
- mudança de configuração.

Campos mínimos:
- audit_id;
- actor;
- action;
- resource;
- case_id;
- timestamp;
- correlation_id;
- result;
- metadata hash.

Logs críticos devem ser append-only.

## 9. Reprodutibilidade da consulta

Registrar:
- authorization snapshot;
- original query;
- query plan;
- retriever versions;
- index versions;
- graph projection version;
- reranker;
- Evidence Pack;
- prompts;
- LLM models;
- parameters;
- final response;
- citations;
- verification outputs.

## 10. Chain of derivation

Answer
→ ReasoningRun
→ EvidencePack
→ RetrievalHit
→ Chunk
→ SourceSpan
→ Page
→ Document
→ SourceFile
→ RAW object hash.

## 11. Retenção e legal hold

Política deve distinguir:
- conteúdo;
- artefatos derivados;
- logs;
- prompts;
- respostas;
- backups.

Legal hold bloqueia exclusão física.

## 12. PII e redaction

Quando aplicável:
- detectar PII;
- classificar;
- redigir para visualização/exportação;
- preservar original autorizado;
- manter log de acesso ao original.

## 13. Multi-tenancy

Baseline: isolamento lógico forte por tenant + case.
Requisitos superiores podem exigir banco/schema/bucket/chaves dedicados por tenant.

## 14. Critérios de aceite

- teste automático de cross-case leakage;
- signed URLs expiram;
- providers recebem apenas dados permitidos;
- cada query possui trilha;
- cada citation chega ao hash do original;
- prompt injection tests fazem parte da suíte;
- secrets nunca aparecem em logs.
