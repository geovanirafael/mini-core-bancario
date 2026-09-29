# 8. Estratégia de testes

> **Estado:** aceite · **Data:** 29 de Setembro de 2026

## Princípios

1. **Pirâmide ajustada ao caso.** Três níveis:
   - **Unitários:** lógica pura, sem base de dados (`Dinheiro`, validações, cálculo do limite diário).
   - **Integração:** *Services* com um **PostgreSQL real**: transacções, bloqueios, `CHECK`, *triggers*.
   - **End-to-end (e2e):** pedidos HTTP à aplicação completa, para provar guards, erros e contrato.

   Neste projecto o nível de integração pesa mais do que numa aplicação típica, porque as regras críticas vivem na base de dados.
2. **Tudo o que toca dinheiro corre contra PostgreSQL real.** *Mocks* ou SQLite não têm `FOR UPDATE`, nem os mesmos `CHECK`, nem os *triggers*: um teste que passa com *mock* e falha em produção dá falsa confiança.
3. **Testar as defesas da base de dados directamente:** tentar escrever `-1` no saldo à mão, tentar `UPDATE` num movimento, tentar uma transferência com origem igual ao destino. A BD tem de recusar.

## Matriz de rastreabilidade

| Requisito | Nível | O que se prova |
|---|---|---|
| RF-01 | e2e | Login válido devolve token; palavra-passe errada → `CREDENCIAIS_INVALIDAS`; token expirado → `NAO_AUTENTICADO`. |
| RF-02, RNF-11 | e2e | Matriz perfil × rota; **uma rota sem token → 401**, testada para *todas* as rotas. |
| RF-03, RF-04 | integração | Conta nasce com saldo 0 e número único; um cliente com duas contas. |
| RF-05 | e2e | Cliente vê só as suas; conta alheia → 404. |
| RF-06 | integração | Saldo sobe, 1 movimento; valor ≤ 0 rejeitado. |
| RF-07 | integração | Saldo insuficiente rejeitado sem alterar nada; **caso-limite:** levantar exactamente o saldo deixa 0. |
| RF-08 | integração | Feliz; saldo insuficiente; mesma conta; destino bloqueado; destino inexistente. |
| RF-09 | integração | Soma do dia; **fronteira da meia-noite em Luanda**. |
| RF-10 | e2e | Filtros de período; cursor sem repetições nem saltos; ordem cronológica. |
| RF-11, RF-12 | integração | Auditoria de sucesso; **falha persiste após *rollback***; auditor lê; ninguém altera. |
| RF-13 | integração | Conta bloqueada não movimenta, nem como origem nem como destino. |
| RF-14 | integração e e2e | Mesma chave → uma só operação; corpo diferente → 409; pedidos paralelos. |
| RF-15, RF-16 | e2e | Só o operador regista clientes; palavra-passe inicial forte; troca de palavra-passe. |
| RNF-01 | integração | Nenhuma coluna guarda palavra-passe legível. |
| RNF-02, RNF-04 | unitário e BD | `Dinheiro` sem vírgula flutuante; `CHECK` recusa saldo negativo. |
| **RNF-05** | concorrência | Ver abaixo. |
| RNF-06, RNF-12 | e2e | Formato de erro uniforme, sem *stack traces*; campo desconhecido no corpo → 400. |
| RNF-10 | e2e | Limitação de pedidos no login. |

## Os testes de concorrência

**Duplo gasto.** Uma conta com 10 000 Kz; disparam-se 20 levantamentos de 1 000 Kz **ao mesmo tempo**. Esperado: **exactamente 10 sucessos e 10 `SALDO_INSUFICIENTE`**, saldo final 0, e a invariante I-6 verdadeira.

**Deadlock.** Muitos pares de transferências cruzadas, A→B e B→A, em paralelo. Esperado: **nenhum erro de *deadlock***, e **conservação do dinheiro** (a soma dos saldos de A e B é a mesma antes e depois).

**Idempotência concorrente.** 10 pedidos simultâneos com a mesma chave. Esperado: uma só operação criada, todas as respostas iguais.

**Teste de sabotagem.** *Um teste que nunca falhou não prova nada.* Depois de o teste passar, desactivam-se **temporariamente** os bloqueios e corre-se de novo: o teste tem de **falhar**. Faz-se manualmente uma vez por teste de concorrência e regista-se o resultado aqui:

| Teste | Sem bloqueio, falhou em | Data |
|---|---|---|
| Duplo gasto | _a preencher_ | |
| Deadlock | _a preencher_ | |
| Idempotência concorrente | _a preencher_ | |

## Tempo e a fronteira da meia-noite

O limite diário conta o dia em Africa/Luanda com timestamps em UTC. Uma transferência às 23:30 UTC já é "dia seguinte" em Luanda (UTC+1). Os testes usam um **relógio controlável** (nunca `Date.now()` directo no código de negócio) e verificam os dois lados da meia-noite.

## Testes de segurança

- **Matriz perfil × rota**, gerada a partir do contrato: para cada rota e perfil, o resultado esperado (2xx, 401 ou 403).
- **BOLA:** o cliente A pede recursos do cliente B → sempre 404.
- ***Mass assignment*:** enviar `perfil` ou `saldoCentimos` no corpo → 400.
- **Login:** a N-ésima tentativa falhada é limitada.

## Organização

- **Dados de teste:** cada teste cria os seus dados através de *fábricas*; testes que dependem de dados deixados por outros são a maior fonte de falhas ao acaso.
- **CI:** a cada *push*, corre *lint*, testes (com um PostgreSQL como serviço) e `npm audit`.

## Decisões

| # | Decisão | Razão |
|---|---|---|
| 1 | Base de dados de teste via **docker-compose** com PostgreSQL dedicado (e não Testcontainers). | Mais simples e suficiente aqui; Testcontainers isola mais mas exige mais configuração. |
| 2 | **Cobertura de cerca de 80%** apenas nos *Services* de contas e transacções; sem meta global. | As percentagens globais incentivam testes vazios; as regras de negócio é que precisam de cobertura. |
| 3 | **Teste de sabotagem manual**, uma vez por teste de concorrência, com resultado registado. | Automatizar não compensa neste prazo. |
