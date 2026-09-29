# ADR-009: Idempotência de transferências por chave

- **Estado:** Aceite
- **Data:** 2026-09-29

## Contexto
Um duplo clique ou uma repetição de pedido (por falha de rede) não pode transferir o dinheiro duas vezes (RF-14).

## Decisão
`POST /transferencias` exige o cabeçalho `Idempotency-Key`. O par (autor, chave) é único na tabela `operacoes`. A mesma chave com o mesmo corpo devolve a resposta original; com corpo diferente devolve 409 (`CHAVE_IDEMPOTENCIA_REUTILIZADA`).

## Alternativas consideradas
Detectar duplicados por conteúdo e janela de tempo: frágil, pode rejeitar transferências legítimas iguais.

## Consequências
- (+) Repetições seguras; a unicidade na BD protege mesmo em pedidos paralelos.
- (−) O cliente da API tem de gerar e guardar a chave.
