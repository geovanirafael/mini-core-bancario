# ADR-004: Saldo guardado na conta, com reconciliação

- **Estado:** Aceite
- **Data:** 2026-09-29

## Contexto
O saldo pode ser calculado a partir dos movimentos ou guardado na conta.

## Decisão
Guardar o saldo na conta (`saldo_cent`) e proteger a consistência com a invariante I-6 (saldo = créditos − débitos), verificada por testes automáticos e por uma consulta de reconciliação.

## Alternativas consideradas
Calcular sempre a partir dos movimentos: nunca há inconsistência, mas somar milhares de movimentos a cada consulta é lento e não há uma linha única onde bloquear.

## Consequências
- (+) Consulta instantânea e uma linha onde bloquear para a concorrência (RNF-05).
- (−) Existe o risco de o saldo e os movimentos divergirem; mitigado pela reconciliação e pelo bloqueio de linhas.
