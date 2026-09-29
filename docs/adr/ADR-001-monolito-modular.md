# ADR-001: Monólito modular

- **Estado:** Aceite
- **Data:** 2026-09-29

## Contexto
Equipa de uma pessoa, prazo curto, e transferências que precisam de uma única transacção ACID (RNF-03).

## Decisão
Uma só aplicação NestJS dividida em módulos com fronteiras claras e sem dependências circulares.

## Alternativas consideradas
Microserviços (por exemplo, um serviço de contas e outro de transacções).

## Consequências
- (+) Uma transacção da base de dados cobre a transferência inteira; deploy simples.
- (−) Se um dia for preciso escalar partes em separado, será preciso extraí-las. Mitigação: fronteiras claras entre módulos.
- Com microserviços, uma transferência entre serviços exigiria transacções distribuídas, muito mais complexas e sem ganho neste âmbito.
