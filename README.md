# GraphRAG Advanced Investigations

Plataforma de inteligência documental para investigações complexas, processos jurídicos extensos e acervos com dezenas de milhares de páginas, anexos, depoimentos, evidências e versões conflitantes.

O objetivo não é construir um "chat com PDF". A plataforma deve transformar um acervo documental em uma base auditável de documentos, entidades, eventos, alegações, evidências e relações, permitindo recuperação híbrida, GraphRAG, comparação temporal, detecção de inconsistências e geração de respostas sustentadas por fontes verificáveis.

## Princípios

1. Evidence-first: o LLM raciocina sobre um pacote de evidências recuperado, não sobre o acervo bruto.
2. Provenance by design: toda unidade derivada preserva vínculo com arquivo, página, posição e hash de origem.
3. Hybrid retrieval: busca lexical, vetorial, por metadados, entidades, timeline e grafo.
4. Human-verifiable: toda conclusão relevante deve ser verificável no documento original.
5. Deterministic orchestration: agentes especializados operam em workflows controlados.
6. Abstention is valid: ausência de evidência não é interpretada como falsidade.
7. Security before retrieval: informação sem autorização não pode entrar no candidate set.
8. Reproducibility: consultas críticas devem ser reproduzíveis com versões de índice, modelo, prompts e evidências.

## Documentação

- [Visão, escopo e requisitos](docs/00-vision-scope-requirements.md)
- [Arquitetura do sistema](docs/01-system-architecture.md)
- [Modelo de domínio e persistência](docs/02-domain-and-data-model.md)
- [Ingestão, OCR e inteligência documental](docs/03-ingestion-document-intelligence.md)
- [Retrieval híbrido, GraphRAG e Evidence Pack](docs/04-retrieval-graphrag.md)
- [Agentes, LLMs e reasoning](docs/05-agents-llms-reasoning.md)
- [Contratos de API e eventos](docs/06-api-events-contracts.md)
- [Segurança, auditoria e reprodutibilidade](docs/07-security-audit-reproducibility.md)
- [Observabilidade, avaliação e qualidade](docs/08-observability-evaluation.md)
- [Infraestrutura e deployment](docs/09-infrastructure-deployment.md)
- [Roadmap: MVP, Produção e Alta Confiabilidade](docs/10-maturity-roadmap.md)
- [Plano de implementação](docs/11-implementation-plan.md)
- [ADRs](docs/adr/)

## Stack de referência

A stack inicial prioriza simplicidade operacional e desacoplamento:

- Python + FastAPI
- PostgreSQL + pgvector
- Object Storage compatível com S3
- Redis
- Worker assíncrono
- OCR/Document Intelligence via provider abstrato
- Embeddings, reranker e LLMs via providers abstratos
- Neo4j a partir da fase de Graph/Evidence Graph
- Temporal ou equivalente para workflows de produção
- OpenTelemetry + Prometheus/Grafana
- Next.js no frontend

A arquitetura evita hardcode de fornecedores. OCR, embeddings, reranking, LLM e armazenamento são acessados por interfaces.

## Estado

Este repositório começa pela especificação técnica. Código de produção deve ser implementado progressivamente conforme o plano definido em `docs/11-implementation-plan.md`.
