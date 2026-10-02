# ADR-002 — Evidence-first RAG

Status: Accepted

## Contexto

RAG convencional pode fornecer ao LLM chunks relevantes porém incompletos, desconectados ou sem relação probatória explícita.

## Decisão

Introduzir Evidence Pack como contrato obrigatório entre Retrieval Plane e Reasoning Plane.

O LLM de reasoning não consulta diretamente o acervo.

## Invariantes

- itens possuem proveniência;
- itens estão autorizados;
- papel da evidência é explícito quando conhecido;
- lacunas permanecem explícitas;
- o pack é versionado e hasheado.

## Consequências

Positivas:
- testabilidade;
- auditabilidade;
- redução de contexto;
- debugging separado entre retrieval e reasoning.

Negativas:
- implementação adicional;
- necessidade de schema/versioning.

## Resultado esperado

Falhas podem ser classificadas como:
- ingestão;
- OCR;
- extraction;
- retrieval;
- reranking;
- evidence assembly;
- reasoning;
- citation validation.
