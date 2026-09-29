# ADR-005: Dinheiro em cêntimos inteiros

- **Estado:** Aceite
- **Data:** 2026-09-29

## Contexto
A vírgula flutuante não representa valores decimais com exactidão (0,1 + 0,2 não dá exactamente 0,3); num banco isso é dinheiro que desaparece.

## Decisão
Guardar todos os valores como inteiros na menor unidade (1 500,50 Kz = 150050), em `bigint` na base de dados, e expô-los assim na API (`valorCentimos`). Toda a aritmética passa pela classe `Dinheiro`.

## Alternativas consideradas
Texto decimal (`"1500.50"`) na API e `numeric` na BD.

## Consequências
- (+) Sem arredondamentos surpresa; sem ambiguidade de formato.
- (−) O driver do PostgreSQL devolve `bigint` como texto: converter num único sítio (*transformer* do TypeORM).
