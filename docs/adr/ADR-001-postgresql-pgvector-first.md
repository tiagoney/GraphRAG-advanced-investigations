# ADR-001 — PostgreSQL + pgvector como baseline

Status: Accepted

## Contexto

O sistema precisa de metadados relacionais, ACL, full-text search e busca vetorial.

Adicionar múltiplos bancos desde o MVP aumenta operação, consistência e troubleshooting.

## Decisão

Usar PostgreSQL como System of Record e pgvector como vector store inicial.

FTS do PostgreSQL será o baseline lexical.

## Consequências

Positivas:
- menos componentes;
- transações;
- filtros por case/ACL próximos ao índice;
- backup simples;
- desenvolvimento rápido.

Negativas:
- pode não ser ótimo para escala extrema;
- tuning HNSW/queries será necessário.

## Gatilhos para vector/search dedicado

Avaliar somente se benchmarks mostrarem:
- latência insuficiente;
- volume além da capacidade operacional;
- recursos de busca especializados indispensáveis.

A migração deve preservar a interface VectorRetriever.
