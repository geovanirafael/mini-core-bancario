# Mini Core Bancário

Núcleo bancário **fictício** e simplificado, construído como projecto de portfólio: contas, depósitos, levantamentos e transferências, com regras que impedem inconsistências (saldo negativo, operações duplicadas ou a meio) e um registo de auditoria permanente.

> **Estado:** em desenvolvimento. O desenho de engenharia está completo e documentado em [`docs/`](docs/); a implementação avança por marcos.

## O problema

Uma instituição financeira precisa garantir que cada kwanza movimentado fica registado de forma consistente e rastreável. Este projecto simula esse núcleo e dá prioridade ao que mais importa num sistema financeiro: **integridade dos dados, concorrência correcta, segurança e auditabilidade**.

## Funcionalidades do MVP

- Autenticação (JWT) com três perfis: cliente, operador e auditor.
- Registo de clientes e abertura de contas (um cliente pode ter várias).
- Depósitos e levantamentos (sem descoberto).
- **Transferências atómicas** entre contas, com bloqueio de linhas, idempotência e limite diário.
- Extracto com filtro por período e paginação por cursor.
- Registo de auditoria imutável de todas as operações sensíveis, incluindo as falhadas.

## Stack

NestJS · TypeScript · PostgreSQL · TypeORM · Docker · Jest

## O que este projecto demonstra

- **Processo de engenharia antes do código:** requisitos testáveis, modelo de domínio com invariantes, decisões registadas em [ADRs](docs/adr/), modelo de ameaças e estratégia de testes.
- **Defesa em profundidade:** as regras críticas (saldo não negativo, imutabilidade do histórico) vivem também na base de dados, não só no código.
- **Concorrência tratada a sério:** bloqueio pessimista ordenado por id e testes que provam a ausência de duplo gasto e de *deadlocks*.

## Como correr

> _A preencher quando o Marco 0 estiver concluído._

```
# 1. Copiar as variáveis de ambiente
# 2. Subir a base de dados
# 3. Aplicar as migrações e arrancar a API
```

A documentação interactiva da API (Swagger UI) ficará disponível quando a aplicação estiver a correr.

## Documentação

Tudo o que foi decidido antes de escrever código está em [`docs/`](docs/README.md): problema e âmbito, requisitos, modelo de domínio, arquitectura, modelo de dados (com diagrama ER), contrato da API, modelo de ameaças, estratégia de testes e plano.

## Limitações conhecidas (riscos residuais)

- Um operador desonesto pode registar depósitos fictícios; não há aprovação em dois níveis.
- Um administrador da base de dados com acesso total pode desactivar o *trigger* de imutabilidade.
- Sem revogação imediata de tokens (não há *refresh tokens*).
- Sem autenticação de dois factores.
- Não é um sistema bancário real nem cumpre requisitos regulatórios.

## Autor

[O seu nome completo] · [LinkedIn / contacto]
