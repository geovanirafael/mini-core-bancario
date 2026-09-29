# ADR-007: Auditoria de falhas fora da transacção de negócio

- **Estado:** Aceite
- **Data:** 2026-09-29

## Contexto
O RF-11 exige rasto de sucessos **e** falhas. Se uma operação falha, a transacção faz *rollback* e tudo o que gravou dentro dela desaparece.

## Decisão
- Sucessos: o registo de auditoria é gravado **dentro** da transacção de negócio (tudo ou nada).
- Falhas: o registo é gravado **fora**, depois do *rollback*.
- Operações falhadas não são guardadas como operações; ficam só na auditoria.

## Alternativas consideradas
Gravar tudo dentro da transacção (perde as falhas); guardar operações com estado "falhada" (suja o histórico financeiro).

## Consequências
- (+) O rasto das falhas é completo e o histórico financeiro fica limpo.
- (−) Uma falha do próprio registo de falhas não é atómica com a operação; aceitável e registada em log.
