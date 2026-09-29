# 1. Problema e âmbito

> **Projecto:** Mini Core Bancário (fictício), nome de trabalho
> **Estado:** aceite · **Data:** 29 de Setembro de 2026

## Declaração do problema

Uma instituição financeira precisa garantir que cada kwanza movimentado fica registado de forma consistente e rastreável. Este projecto simula esse núcleo: gestão de contas, depósitos, levantamentos e transferências, com regras que impedem inconsistências (saldo negativo, operações duplicadas ou a meio) e um registo permanente de tudo o que acontece.

## Actores

| Actor | O que precisa |
|---|---|
| **Cliente** | Consultar o saldo e o extracto das suas contas e fazer transferências. |
| **Operador (balcão)** | Registar clientes, abrir contas e registar depósitos e levantamentos. |
| **Auditor** | Acesso só de leitura ao registo de auditoria, para verificar quem fez o quê e quando. |

## Âmbito do MVP

1. Autenticação e perfis de acesso (cada actor só vê e faz o que lhe compete).
2. Registo de clientes, abertura e gestão de contas.
3. Depósitos e levantamentos.
4. Transferência entre contas, atómica (ou acontece por inteiro, ou não acontece).
5. Extracto com filtro por período.
6. Registo de auditoria de todas as operações sensíveis.

## Não-objectivos

- Várias moedas e câmbio.
- Juros, crédito e cartões.
- Integração com sistemas ou pagamentos reais.
- Notificações (SMS, email).
- Aplicação mobile e frontend próprio (a demonstração faz-se pelo Swagger UI).
- Aprovações em vários níveis.
- Verificação de identidade real (KYC).
- Deploy em produção.

## Critério de sucesso

Uma demonstração de 5 minutos que percorre o fluxo completo: criar conta → depositar → transferir → ver extracto → auditar.

## Decisões de âmbito

| # | Decisão | Razão |
|---|---|---|
| 1 | Moeda única (kwanza). | Multi-moeda exige taxas de câmbio, arredondamentos e mais regras, com pouco valor para o objectivo. |
| 2 | O cliente faz transferências sozinho; o operador trata de clientes, contas, depósitos e levantamentos. | A transferência (o caso mais rico em SQL transaccional) fica exposta via API; o balcão fica com o que só um funcionário faria. |
| 3 | O auditor é um perfil separado. | Mostra separação de responsabilidades com pouco esforço extra. |
