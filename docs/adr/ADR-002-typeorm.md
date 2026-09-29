# ADR-002: TypeORM como ORM

- **Estado:** Aceite
- **Data:** 2026-09-29

## Contexto
O requisito mais valioso do projecto (RNF-05, concorrência) exige bloqueio de linhas (`SELECT ... FOR UPDATE`) e transacções explícitas.

## Decisão
Usar TypeORM, que suporta bloqueio pessimista e transacções explícitas de forma nativa e encaixa bem com o estilo do NestJS (decorators e entidades).

## Alternativas consideradas
Prisma: modelo legível e migrações agradáveis, mas, até onde se sabe, sem bloqueio de linhas nativo, o que obrigaria a SQL cru na transferência. Confirmar na documentação actual antes de reavaliar.

## Consequências
- (+) O bloqueio pessimista fica expresso na própria ferramenta.
- (−) Menos ergonomia de tipos do que o Prisma; as migrações com `CHECK` e *triggers* são SQL escrito à mão.
