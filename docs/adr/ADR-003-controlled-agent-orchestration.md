# ADR-003 — Orquestração controlada de agentes

Status: Accepted

## Contexto

Agentes autônomos podem introduzir loops, custos imprevisíveis, tool calls inadequados e baixa reprodutibilidade.

## Decisão

Usar agentes especializados dentro de workflows/state machines.

Cada agente possui:
- input schema;
- output schema;
- ferramentas permitidas;
- budget;
- timeout;
- retry;
- audit trail.

## Consequências

Positivas:
- previsibilidade;
- observabilidade;
- segurança;
- testabilidade.

Negativas:
- menos flexibilidade espontânea;
- mais desenho explícito de workflows.

## Exceção

Experimentação com agente mais autônomo pode ocorrer em ambiente isolado, sem acesso privilegiado e fora do caminho de produção.
