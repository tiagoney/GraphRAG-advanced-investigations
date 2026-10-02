# 08 — Observabilidade, Avaliação e Qualidade

## 1. Dimensões

A qualidade deve ser medida separadamente em três planos:

1. qualidade documental;
2. qualidade de retrieval/reasoning;
3. qualidade operacional.

## 2. Telemetria

Usar OpenTelemetry para traces distribuídos.

Propagar:
- trace_id;
- correlation_id;
- query_run_id;
- processing_run_id;
- case_id pseudonimizado quando necessário.

## 3. Métricas de ingestão

- files_received_total;
- bytes_ingested_total;
- pages_processed_total;
- extraction_latency;
- ocr_latency;
- ocr_failure_rate;
- low_confidence_page_rate;
- queue_depth;
- retries_total.

## 4. Métricas de embeddings e índices

- chunks_total;
- embeddings_total;
- embedding_latency;
- embedding_cost;
- index_build_duration;
- index_freshness;
- failed_embeddings;
- graph_projection_lag.

## 5. Métricas de retrieval

- Recall@K;
- Precision@K;
- MRR;
- nDCG;
- lexical hit contribution;
- vector hit contribution;
- graph hit contribution;
- reranker lift;
- evidence_pack_size;
- retrieval latency.

## 6. Métricas semânticas e de resposta

- entity resolution precision/recall/F1;
- claim extraction precision/recall/F1;
- event extraction precision/recall/F1;
- contradiction detection precision/recall;
- citation precision;
- citation coverage;
- grounded claim ratio;
- unsupported claim rate;
- answer completeness;
- abstention quality.

## 7. Gold dataset

Criar dataset por domínio contendo:
- case snapshot;
- pergunta;
- intent;
- gold documents;
- gold spans;
- expected entities;
- expected claims;
- expected relations;
- fatos aceitáveis na resposta;
- conclusões não suportadas que o sistema não deve produzir.

Separar:
- development set;
- validation set;
- locked test set.

## 8. Evaluation harness

Cada alteração relevante deve poder executar:
- retrieval-only benchmark;
- extraction benchmark;
- end-to-end benchmark;
- regression comparison.

Persistir:
- git SHA;
- config hash;
- model versions;
- dataset version;
- metrics;
- timestamp.

## 9. Qualidade de OCR

Quando houver ground truth:
- CER;
- WER;
- table cell accuracy;
- layout block accuracy.

Também medir impacto do OCR no retrieval.

## 10. Testes de contradição

Dataset deve conter:
- contradição explícita;
- diferença temporal legítima;
- mudança de condição;
- negação;
- ausência de informação;
- testemunhos compatíveis;
- múltiplas fontes derivadas da mesma origem.

Objetivo: reduzir falso positivo de contradição.

## 11. Testes de segurança

- cross-tenant;
- cross-case;
- metadata leakage;
- vector leakage;
- graph leakage;
- prompt injection;
- malicious PDF;
- oversized files;
- zip bomb;
- replay de signed URL.

## 12. SLOs

Definir metas após benchmark real.

Medir desde o MVP:
- disponibilidade da Query API;
- P50/P95/P99 de query;
- pages/min na ingestão;
- freshness do índice;
- provider error rate.

Não transformar target provisório em garantia antes de testes de carga.

## 13. Dashboards

Painéis mínimos:
- ingestion health;
- provider health;
- retrieval quality;
- model cost;
- security/audit anomalies;
- case processing status.

## 14. Release gate

Nova versão de retrieval, prompt ou modelo não entra em produção apenas por avaliação subjetiva.

Exigir:
- benchmark;
- comparação de regressão;
- análise dos piores casos;
- security suite;
- reproducibility test.
