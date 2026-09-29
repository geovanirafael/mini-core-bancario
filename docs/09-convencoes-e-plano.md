# 9. Convenções, repositório e planeamento

> **Estado:** aceite · **Data:** 29 de Setembro de 2026

## Estrutura do repositório

```
mini-core-bancario/
├─ docs/                  (documentos, ADRs e diagrama ER)
│  └─ adr/
├─ backend/
│  ├─ src/
│  │  ├─ auth/  users/  accounts/  transactions/  audit/
│  │  ├─ common/          (Dinheiro, filtros, decorators)
│  │  ├─ config/  database/   (migrações aqui)
│  │  └─ main.ts
│  └─ test/               (integração e e2e)
├─ docker-compose.yml     (PostgreSQL de desenvolvimento e de teste)
├─ .env.example
└─ README.md
```

Dentro de cada módulo: *controller*, *service*, `dto/`, `entities/`, respeitando a regra de que o *service* não conhece HTTP.

## Convenções técnicas

- **TypeScript em modo estrito**, com ESLint e Prettier.
- **Migrações, nunca `synchronize`.** O `synchronize: true` do TypeORM pode alterar ou apagar colunas sem aviso. O esquema nasce de **migrações versionadas**, e é nelas que entram os `CHECK` e os *triggers* (SQL puro).
- **Configuração validada no arranque** (*fail fast*): se falta uma variável obrigatória, a aplicação recusa arrancar.
- **Nomes do domínio em português**, comentários apenas onde explicam um *porquê*.

## Git

- **`main` sempre funcional.** Ramos curtos (`feat/transferencia`) fundidos por *pull request*, mesmo a trabalhar sozinho, com uma pequena lista de verificação.
- **Conventional Commits** com o requisito referenciado: `feat(transacoes): transferência atómica (RF-08)`. Tipos: `feat`, `fix`, `test`, `docs`, `chore`.
- **Etiquetas por marco** (`v0.1.0`, `v0.2.0`).

## Definition of Done

Um requisito só se considera concluído quando:

1. o código cumpre os critérios de aceitação e os **testes passam**;
2. o *lint* está limpo e não há segredos no código;
3. a operação regista auditoria onde o desenho o exige;
4. a documentação OpenAPI reflecte a rota;
5. o commit ou PR referencia o requisito.

## O README

Estrutura: o que é e o problema que resolve; como correr em três comandos; link para `docs/`; as decisões principais e os **riscos residuais**. Ver o modelo em [`../README.md`](../README.md).

## Planeamento até 5 de Outubro de 2026

Regra: **primeiro o essencial e o que mais impressiona**; sem frontend.

| Marco | Dia | Conteúdo |
|---|---|---|
| 0 | 29–30 Set | Repositório, `docs/` no Git, docker-compose, esqueleto NestJS, migração inicial com **todas as tabelas, CHECKs e triggers** |
| 1 | 1 Out | Autenticação, guards por perfil (protegido por omissão), seed de operador e auditor, registo de clientes (RF-15) |
| 2 | 2 Out | Contas, depósitos, levantamentos, classe `Dinheiro`, auditoria de sucesso; testes de integração |
| 3 | 3 Out | **Transferência** com bloqueios, idempotência e limite diário; testes de concorrência |
| 4 | 4 Out | Extracto, consulta de auditoria, Swagger, README, polir e CV |
| — | 5 Out | Margem de segurança |

**Estratégia de candidatura:** submeter por volta de 3 ou 4 de Outubro, com o projecto identificado como "em desenvolvimento" e o link do repositório com a documentação, em vez de esperar pelo último dia. Confirmar o prazo real em careers.bfa.ao (o anúncio indicava duas datas diferentes).

## Ordem de corte se o tempo apertar

Do primeiro a sair para o último: **RF-16**, **RF-13**, **RF-09**, **RF-14**. Nunca se corta o que é *Must*, nem os testes de concorrência.

## Decisões

| # | Decisão | Razão |
|---|---|---|
| 1 | Ramos curtos com *pull request* e Conventional Commits. | Custa minutos por funcionalidade e dá um histórico legível para um recrutador. |
| 2 | Sem frontend no MVP; demonstração pelo Swagger UI. | Os dias poupados fazem mais falta nos testes e na concorrência. |
| 3 | Ordem de corte acima. | Decidir agora evita improvisar sob pressão. |
