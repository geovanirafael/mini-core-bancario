# ADR-006: Bloqueio pessimista de linhas, ordenado por id

- **Estado:** Aceite
- **Data:** 2026-09-29

## Contexto
Duas operações simultâneas na mesma conta não podem gastar mais do que o saldo (RNF-05). Transferências cruzadas (A→B e B→A) arriscam *deadlock*.

## Decisão
Em cada operação que altera saldos, bloquear as contas envolvidas com `SELECT ... FOR UPDATE`, **sempre por ordem crescente de id**, e só depois validar as regras (saldo, limite diário, estados).

## Alternativas consideradas
Bloqueio optimista (coluna de versão): deixa ambas as operações avançar e falha uma na gravação; menos previsível em contas muito movimentadas.

## Consequências
- (+) Correcção previsível; a ordem por id evita *deadlocks* entre transferências cruzadas.
- (−) Operações na mesma conta são serializadas; aceitável neste âmbito.
- Verificado pelos testes de concorrência (documento 8).
