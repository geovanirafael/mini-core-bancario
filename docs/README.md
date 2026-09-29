# Documentação do projecto

Documentos de engenharia produzidos **antes** de escrever código, por ordem. Cada um alimenta o seguinte.

| # | Documento | O que responde |
|---|---|---|
| 1 | [Problema e âmbito](01-problema-e-ambito.md) | O que resolvemos, para quem e o que fica de fora |
| 2 | [Requisitos](02-requisitos.md) | O que o sistema faz e como se comporta, de forma testável |
| 3 | [Modelo de domínio](03-modelo-de-dominio.md) | Entidades, relações, invariantes e ciclos de vida |
| 4 | [Arquitectura](04-arquitectura.md) | Como o NestJS organiza o sistema em módulos |
| 5 | [Modelo de dados](05-modelo-de-dados.md) | Tabelas, restrições, índices, diagrama ER e protocolo de concorrência |
| 6 | [Contrato da API](06-contrato-api.md) | Rotas, formatos, erros e convenções |
| 7 | [Segurança e ameaças](07-modelo-de-ameacas.md) | O que pode correr mal e onde é travado |
| 8 | [Estratégia de testes](08-estrategia-de-testes.md) | Como cada requisito e invariante se prova |
| 9 | [Convenções e plano](09-convencoes-e-plano.md) | Estrutura, Git, Definition of Done e marcos |

## Registo de decisões de arquitectura (ADRs)

| ADR | Decisão |
|---|---|
| [001](adr/ADR-001-monolito-modular.md) | Monólito modular |
| [002](adr/ADR-002-typeorm.md) | TypeORM como ORM |
| [003](adr/ADR-003-jwt-curto-sem-refresh.md) | JWT de acesso curto, sem *refresh tokens* |
| [004](adr/ADR-004-saldo-guardado-com-reconciliacao.md) | Saldo guardado na conta, com reconciliação |
| [005](adr/ADR-005-dinheiro-em-centimos-inteiros.md) | Dinheiro em cêntimos inteiros |
| [006](adr/ADR-006-bloqueio-pessimista-ordenado.md) | Bloqueio pessimista de linhas, ordenado por id |
| [007](adr/ADR-007-auditoria-de-falhas-fora-da-transaccao.md) | Auditoria de falhas fora da transacção |
| [008](adr/ADR-008-imutabilidade-por-trigger.md) | Imutabilidade por *trigger* |
| [009](adr/ADR-009-idempotencia-por-chave.md) | Idempotência de transferências por chave |
| [010](adr/ADR-010-fuso-africa-luanda.md) | Timestamps em UTC, "dia" em Africa/Luanda |
| [011](adr/ADR-011-identificadores-hibridos.md) | Identificadores híbridos (UUID e sequencial) |
