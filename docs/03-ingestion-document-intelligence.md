# 03 — Ingestão, OCR e Inteligência Documental

## 1. Objetivo

Converter arquivos heterogêneos em uma representação documental normalizada sem perder proveniência.

## 2. Pipeline

1. Receive
2. Validate
3. Hash
4. Persist RAW
5. Detect type
6. Extract container/anexos
7. Choose extraction strategy
8. Native extraction ou OCR
9. Layout analysis
10. Quality assessment
11. Normalize
12. Build page/block model
13. Persist derived artifacts
14. Emit `DocumentStructured`

## 3. Upload

O Ingestion Gateway deve:
- usar streaming;
- impedir execução de conteúdo;
- validar tamanho;
- detectar MIME por conteúdo, não apenas extensão;
- calcular hash durante streaming;
- fazer malware scan quando exigido;
- registrar user/case/origin.

Arquivos duplicados:
- o sistema pode deduplicar fisicamente;
- não deve deduplicar logicamente sem preservar cada contexto de origem.

## 4. Object Storage

Buckets/prefixos recomendados:

- `raw/`
- `normalized/`
- `pages/`
- `ocr/`
- `layout/`
- `derived/`
- `transcripts/`
- `manifests/`

RAW deve ser imutável em ambientes de alta confiabilidade.

## 5. Routing de extração

### Documento nativo

Preferir:
1. parser nativo;
2. validação de cobertura textual;
3. fallback seletivo para OCR em páginas sem texto ou com texto corrompido.

### Documento digitalizado

Aplicar:
- orientation detection;
- deskew;
- denoise quando necessário;
- OCR;
- layout extraction;
- confidence evaluation.

### Documento híbrido

Processamento por página.

## 6. Interface de OCR

Contrato conceitual:

`DocumentIntelligenceProvider.analyze(document) -> DocumentAnalysisResult`

Resultado deve incluir:
- pages;
- text;
- spans;
- paragraphs;
- tables;
- figures;
- key-values quando disponíveis;
- confidence;
- provider metadata.

A camada de domínio não conhece o provider.

## 7. Quality Gate

Por página calcular:
- percentual de caracteres confiáveis;
- palavras de baixa confiança;
- densidade textual;
- divergência native-vs-OCR quando ambos existirem;
- presença de artefatos;
- suspeita de ordem de leitura incorreta.

Política exemplo:
- HIGH: segue automaticamente;
- MEDIUM: segue com flag;
- LOW: segundo método;
- CRITICAL: revisão humana.

Thresholds devem ser configuráveis e calibrados por benchmark.

## 8. OCR Ensemble — alta confiabilidade

Para páginas críticas ou de baixa confiança:

1. executar OCR A;
2. executar OCR B ou vision model;
3. alinhar spans;
4. comparar divergências;
5. selecionar resultado por confidence/regras;
6. se divergência persistir, criar `HumanReviewTask`.

Nunca substituir silenciosamente o OCR original; manter versões.

## 9. Tabelas

Tabelas devem ser armazenadas em:
- estrutura de linhas/colunas;
- texto linearizado para retrieval;
- vínculo com células e spans originais.

Valores numéricos relevantes não devem depender apenas de embeddings.

## 10. Áudio e vídeo

Pipeline opcional:
- extract audio;
- speech-to-text;
- diarization;
- timestamps;
- speaker candidates;
- human confirmation de identidade;
- transcript spans.

Cada trecho deve preservar `start_ms/end_ms`.

## 11. Chunking

O chunker deve considerar:
- heading;
- parágrafo;
- tabela;
- lista;
- depoimento/pergunta-resposta;
- troca de interlocutor;
- limite de tokens.

Evitar cortar:
- pergunta e resposta;
- cláusula contratual no meio;
- linha de tabela fora do cabeçalho;
- assinatura de seu contexto.

## 12. Parent-child retrieval

Criar representações:
- child chunk: granular para matching;
- parent section: contexto expandido;
- document summary: apenas para descoberta, nunca substitui evidência primária.

## 13. Critérios de aceite

- 100% dos documentos têm SHA-256.
- 100% dos chunks selecionáveis têm proveniência.
- páginas com OCR abaixo de threshold são sinalizadas.
- reprocessamento não duplica documento lógico.
- falha em uma página pode ser retomada sem reprocessar todo o caso.
- original RAW nunca é sobrescrito.
