# 00 — Visão, Escopo e Requisitos

## 1. Problema

Processos jurídicos e investigações documentais podem possuir dezenas de milhares de páginas e múltiplos anexos: petições, laudos, e-mails, contratos, planilhas, imagens, transcrições, depoimentos, áudios e versões diferentes do mesmo fato.

Um RAG convencional baseado apenas em chunk + embedding é insuficiente quando a pergunta exige:

- comparar declarações da mesma pessoa em momentos distintos;
- correlacionar fala, documento, evento e prova;
- identificar inconsistências ou convergências;
- reconstruir cronologia;
- diferenciar alegação, evidência e fato documental;
- explicar quais fontes sustentam cada conclusão;
- manter rastreabilidade até página e posição exata;
- operar com alto recall sem perder precisão.

## 2. Objetivo do sistema

Construir uma plataforma capaz de:

1. ingerir acervos extremamente grandes;
2. preservar cadeia de custódia e proveniência;
3. extrair conteúdo textual e estrutural;
4. identificar entidades, eventos, alegações e evidências;
5. representar relações em banco relacional e grafo;
6. executar recuperação híbrida multiestágio;
7. produzir um Evidence Pack controlado;
8. usar LLMs para análise somente depois da recuperação;
9. verificar citações e suporte factual antes de responder;
10. registrar toda a execução para auditoria e reprodução.

## 3. Escopo funcional

### RF-001 — Gestão de casos

O sistema deve permitir criar um `Case` como unidade lógica de investigação.

Cada caso deve possuir:
- identificador imutável;
- título;
- descrição;
- classificação de confidencialidade;
- participantes autorizados;
- status;
- datas de criação e encerramento;
- política de retenção.

### RF-002 — Ingestão

Deve aceitar, inicialmente:
- PDF nativo;
- PDF digitalizado;
- DOCX;
- XLSX/CSV;
- imagens;
- arquivos compactados;
- e-mail exportado e anexos;
- áudio/vídeo, quando habilitado.

### RF-003 — Cadeia de custódia digital

Para cada arquivo:
- calcular SHA-256 antes de qualquer transformação;
- registrar origem e data de ingestão;
- manter original imutável;
- registrar cada artefato derivado;
- manter relação pai/filho entre anexos;
- versionar processamentos.

### RF-004 — Document Intelligence

Deve produzir:
- texto;
- páginas;
- blocos;
- parágrafos;
- tabelas;
- figuras;
- coordenadas/bounding boxes quando disponíveis;
- confiança por elemento;
- idioma;
- ordem de leitura.

### RF-005 — Segmentação jurídica

Chunks devem respeitar fronteiras estruturais. O sistema deve preservar hierarquia:

Case → SourceFile → Document → Section → Page → Block → Chunk → Span.

### RF-006 — Extração semântica

Deve extrair, com score de confiança e proveniência:
- pessoas;
- organizações;
- locais;
- datas;
- valores;
- documentos citados;
- eventos;
- alegações/claims;
- evidências;
- relações.

### RF-007 — Resolução de entidades

Deve tratar aliases e homônimos.

Exemplo:
"João Silva", "Dr. João", "J. A. Silva" não devem ser fundidos automaticamente sem evidência suficiente.

### RF-008 — Timeline

Eventos devem possuir:
- início/fim;
- precisão temporal;
- participantes;
- local;
- fonte;
- evidências relacionadas;
- claims relacionados;
- confidence score.

### RF-009 — Recuperação híbrida

A busca deve poder combinar:
- lexical/BM25 ou full-text;
- vetor/dense retrieval;
- filtros por metadados;
- entidades;
- timeline;
- graph traversal;
- claims;
- evidências.

### RF-010 — Reranking

Resultados agregados devem ser reranqueados antes de compor o contexto final.

### RF-011 — Evidence Pack

Nenhum LLM analítico deve receber diretamente o acervo completo. Deve receber um pacote estruturado contendo:
- questão original;
- subquestões;
- entidades resolvidas;
- candidatos;
- fontes;
- claims;
- evidências favoráveis;
- evidências divergentes;
- timeline relevante;
- scores;
- lacunas.

### RF-012 — Análise

O sistema deve suportar:
- localização factual;
- comparação de versões;
- detecção de inconsistências;
- corroboração;
- reconstrução cronológica;
- análise de relações;
- síntese documental;
- análise jurídica, quando explicitamente habilitada.

### RF-013 — Respostas citadas

Cada afirmação factual relevante deve possuir referência para:
- documento;
- página;
- span;
- opcionalmente bounding box;
- hash da origem.

### RF-014 — Abstention

Quando evidência for insuficiente, o sistema deve indicar insuficiência, em vez de completar lacunas por plausibilidade.

### RF-015 — Revisão humana

O sistema deve permitir:
- aprovar/corrigir entidade;
- aprovar/corrigir claim;
- marcar relação inválida;
- revisar OCR;
- validar evidência;
- registrar responsável pela revisão.

## 4. Requisitos não funcionais

### RNF-001 — Escalabilidade

Meta inicial por caso:
- 100.000 páginas;
- milhões de blocos;
- centenas de milhares a milhões de chunks.

A arquitetura não deve assumir que todo o caso cabe em memória ou contexto LLM.

### RNF-002 — Idempotência

Todos os jobs de ingestão e processamento devem poder ser repetidos sem duplicação lógica.

### RNF-003 — Segurança

Autorização deve ser aplicada antes do retrieval.

### RNF-004 — Auditabilidade

Toda resposta deve possuir `query_run_id` e `evidence_pack_id`.

### RNF-005 — Reprodutibilidade

Execuções críticas devem registrar:
- versão do código;
- versão do prompt;
- modelos;
- configuração;
- versão dos índices;
- IDs das evidências.

### RNF-006 — Resiliência

Falha de OCR, embedding ou LLM em um arquivo não deve invalidar o caso inteiro.

### RNF-007 — Observabilidade

Jobs e consultas devem emitir logs estruturados, métricas e traces.

### RNF-008 — Provider independence

OCR, embeddings, reranking, LLM, vector store e object storage devem possuir interfaces abstratas.

## 5. Fora de escopo inicial

- decisão judicial automática;
- classificação definitiva de culpa, dolo ou mentira;
- substituição de revisão humana;
- inferência de fatos sem fonte verificável;
- agentes totalmente autônomos com acesso irrestrito ao acervo.

## 6. Critérios globais de aceite

Uma versão não pode ser considerada pronta para uso jurídico sério se:
1. não houver proveniência por página/span;
2. retrieval não for medido com benchmark;
3. ACL puder ser aplicada apenas depois do retrieval;
4. citações não forem verificáveis;
5. execução não puder ser reproduzida;
6. o sistema apresentar inferência como fato documental.
