# ADR-010: Timestamps em UTC, "dia" do limite diário em Africa/Luanda

- **Estado:** Aceite
- **Data:** 2026-09-29

## Contexto
O limite diário (RF-09, I-8) conta transferências "por dia". Angola está em UTC+1.

## Decisão
Guardar todos os instantes em UTC (`timestamptz`) e calcular o "dia" do limite no fuso Africa/Luanda.

## Alternativas consideradas
Contar o dia em UTC: o limite reiniciaria à 01:00 da manhã em Angola.

## Consequências
- (+) Comportamento coerente para quem está em Angola.
- (−) A fronteira da meia-noite exige testes com relógio controlável (documento 8).
