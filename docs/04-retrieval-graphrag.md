# 04 — Retrieval Híbrido, GraphRAG e Evidence Pack

## 1. Objetivo

Maximizar recall na geração de candidatos e precisão antes do reasoning.

O retrieval é multiestágio:

Query Understanding → Decomposition → Parallel Retrieval → Fusion → Reranking → Context Expansion → Evidence Pack.

## 2. Query Understanding

Classificar:
- FACT_LOOKUP;
- DOCUMENT_LOOKUP;
- ENTITY_HISTORY;
- TIMELINE;
- CLAIM_COMPARISON;
- CONTRADICTION;
- CORROBORATION;
- RELATIONSHIP;
- SUMMARY;
- LEGAL_ANALYSIS.

Extrair:
- entidades;
- documentos;
- intervalos de data;
- identificadores exatos;
- conceitos;
- restrições.

## 3. Query decomposition

Exemplo:
"Carlos mudou a versão sobre a assinatura do contrato?"

Subconsultas:
1. resolver entidade Carlos;
2. resolver contrato;
3. buscar claims atribuídos a Carlos;
4. filtrar claims sobre assinatura;
5. ordenar por asserted_at/event_time;
6. buscar evidência documental independente;
7. comparar proposições;
8. buscar relações de contradição/corroboration.

O plano deve ser persistido.

## 4. Canais de retrieval

### 4.1 Lexical

Para:
- nomes;
- frases;
- identificadores;
- números;
- artigos;
- datas;
- texto exato.

MVP: PostgreSQL FTS + trigram.

Produção: avaliar OpenSearch/Elasticsearch apenas com evidência de necessidade.

### 4.2 Dense vector

Para semântica e paráfrases.

MVP: pgvector HNSW.

Embeddings devem possuir:
- embedding_model;
- embedding_version;
- dimensions;
- input_hash;
- created_at.

### 4.3 Metadata

Filtros:
- case_id obrigatório;
- ACL;
- document_type;
- date;
- author;
- source;
- page range;
- entity.

### 4.4 Entity retrieval

Buscar menções e documentos conectados a entidades resolvidas e candidates.

### 4.5 Timeline retrieval

Buscar eventos por intervalo, participante, local e tipo.

### 4.6 Graph retrieval

Executar traversal restrito e explicável.

Exemplos:
- Person → STATED → Claim;
- Claim ← CONTRADICTS → Evidence;
- Event ← PARTICIPATED_IN ← Person.

Traversal deve possuir limite de hops e limite de nós.

## 5. Candidate generation

Cada canal retorna `RetrievalCandidate`:
- source_id;
- source_type;
- source_span_id;
- retriever;
- raw_score;
- rank;
- metadata;
- authorization_context.

Alvo inicial por canal: configurável, tipicamente 20–200.

## 6. Fusion

Usar Reciprocal Rank Fusion como baseline.

A fórmula/configuração deve ser versionada.

Evitar normalizar scores heterogêneos de forma ad hoc.

## 7. Deduplicação

Candidatos duplicados ou altamente sobrepostos:
- consolidar;
- preservar quais retrievers encontraram o item;
- somar evidência de descoberta sem duplicar contexto.

## 8. Reranking

Interface:
`Reranker.rerank(query, candidates) -> RankedCandidates`

Aplicar apenas após candidate generation.

Guardar:
- model;
- version;
- score;
- input set;
- latency.

## 9. Context expansion

Para candidatos finais, recuperar:
- chunks vizinhos;
- parent section;
- document metadata;
- speaker context;
- headings;
- tabela completa quando match ocorreu em célula.

A expansão não deve quebrar ACL.

## 10. Knowledge Graph

Nós iniciais:
- Case;
- Document;
- Person;
- Organization;
- Claim;
- Evidence;
- Event;
- Location;
- Contract;
- Asset;
- Communication.

Relações:
- MENTIONS;
- AUTHORED_BY;
- SIGNED_BY;
- STATED_BY;
- PARTICIPATED_IN;
- OCCURRED_AT;
- OCCURRED_ON;
- SUPPORTS;
- CONTRADICTS;
- QUALIFIES;
- REFERS_TO;
- ATTACHED_TO;
- SAME_AS;
- PRECEDES;
- FOLLOWS.

## 11. Graph projection

PostgreSQL é a origem dos objetos semânticos.

Neo4j é projeção reconstruível.

Cada nó/edge deve carregar:
- source_record_id;
- case_id;
- projection_version;
- provenance quando aplicável.

## 12. Evidence Pack

Schema lógico:

- `query`
- `query_plan`
- `resolved_entities`
- `time_constraints`
- `supporting_items[]`
- `contradicting_items[]`
- `context_items[]`
- `timeline[]`
- `claims[]`
- `graph_paths[]`
- `open_questions[]`
- `retrieval_diagnostics`

Cada item inclui:
- source type/id;
- exact span;
- document/page;
- text;
- role;
- scores;
- provenance;
- authorization proof/token reference.

## 13. Evidence Pack invariants

1. Nenhum item sem case_id.
2. Nenhum item sem autorização.
3. Nenhum item textual sem SourceSpan.
4. Itens gerados por LLM não podem substituir fonte primária.
5. Resumos devem apontar para os itens usados.
6. `pack_hash` deve ser calculado após serialização canônica.

## 14. RAG anti-patterns proibidos

- top-k vetorial único como única estratégia;
- chunk sem metadados;
- embedding de página sem proveniência;
- enviar milhares de chunks diretamente ao LLM;
- usar resumo gerado como evidência;
- graph edge sem fonte quando a relação exige evidência;
- citar documento sem página/span;
- pós-filtrar ACL depois da busca.

## 15. Critérios de aceite

Antes de produção:
- Recall@K medido em conjunto gold;
- lexical e vector comparados isoladamente;
- hybrid demonstra ganho ou justificativa;
- reranker demonstra ganho;
- citation hit rate medido;
- queries de contradição e timeline possuem testes específicos.
