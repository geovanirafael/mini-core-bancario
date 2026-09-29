# ADR-011: Identificadores híbridos (UUID e sequencial)

- **Estado:** Aceite
- **Data:** 2026-09-29

## Contexto
Os identificadores expostos na API não devem ser adivinháveis (risco de enumeração e *BOLA*), mas as tabelas só de acrescentar beneficiam de ordem estável.

## Decisão
UUID para utilizadores, clientes, contas e operações. Inteiro sequencial (`bigint` identidade) para movimentos e registos de auditoria.

## Alternativas consideradas
Inteiros em tudo (fáceis de enumerar); UUID em tudo (sem ordem estável nas tabelas de acrescentar).

## Consequências
- (+) Menos superfície para enumeração; ordenação e paginação por cursor estáveis onde interessa.
- (−) Dois estilos de identificador no modelo, que é preciso manter consistentes no código.
