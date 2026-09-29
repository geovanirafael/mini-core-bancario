# 6. Contrato da API

> **Estado:** aceite · **Data:** 29 de Setembro de 2026

**Contract-first:** o contrato é definido antes do código. O NestJS gera depois a documentação OpenAPI a partir do código, e verifica-se que o código cumpre o que está aqui. Se divergir, **o contrato ganha**.

## Princípios

- **Recursos são substantivos** (`contas`, `transferencias`); o verbo é o método HTTP. Acções que não são CRUD modelam-se como criação de um recurso (transferir = criar uma transferência).
- **400 vs 422:** *400* é pedido mal formado; *422* é pedido bem formado que viola uma regra de negócio.
- **Não revelar existência:** se um cliente pede uma conta que não é dele, a resposta é **404**, não 403 (com 403 confirmaríamos que a conta existe).
- **Versão no prefixo:** `/api/v1`.

## Rotas

Todas com o prefixo `/api/v1`. Autenticação por `Authorization: Bearer <token>`, excepto no login e na saúde.

| Rota | Quem | Requisito | Sucesso |
|---|---|---|---|
| `POST /auth/login` | público | RF-01 | 200 |
| `PATCH /auth/palavra-passe` | autenticado | RF-16 | 204 |
| `POST /clientes` | operador | RF-15 | 201 |
| `POST /contas` | operador | RF-03, RF-04 | 201 |
| `GET /contas` | cliente (as suas); operador (todas, filtro `clienteId`) | RF-05 | 200 |
| `GET /contas/{id}` | cliente (própria); operador | RF-05 | 200 |
| `PATCH /contas/{id}` (estado, limite diário) | operador | RF-09, RF-13 | 200 |
| `POST /contas/{id}/depositos` | operador | RF-06 | 201 |
| `POST /contas/{id}/levantamentos` | operador | RF-07 | 201 |
| `POST /transferencias` | cliente | RF-08, RF-14 | 201 |
| `GET /contas/{id}/movimentos` | cliente (própria) | RF-10 | 200 |
| `GET /auditoria` | auditor | RF-11, RF-12 | 200 |
| `GET /saude` | público | operacional | 200 |

## Convenções transversais

- **Idempotência (RF-14):** `POST /transferencias` exige o cabeçalho `Idempotency-Key`. O mesmo pedido repetido com a mesma chave devolve a resposta original sem duplicar; a mesma chave com corpo diferente é rejeitada (409).
- **Paginação por cursor** no extracto e na auditoria: parâmetros `limite` (com máximo definido) e `cursor`. O extracto aceita `de` e `ate` (datas); a auditoria aceita ainda `autorId`, `accao` e `resultado`.
- **Valores monetários** são sempre inteiros em cêntimos (`valorCentimos`).
- **Nomes em português** nas rotas e campos, coerentes com a linguagem do domínio.

## Exemplo: transferência

Pedido (com o cabeçalho `Idempotency-Key` preenchido):

```json
{
  "contaOrigemId": "b1f0…",
  "numeroContaDestino": "0012345678",
  "valorCentimos": 300000
}
```

Resposta **201**:

```json
{
  "id": "7c9e…",
  "tipo": "transferencia",
  "valorCentimos": 300000,
  "numeroContaDestino": "0012345678",
  "saldoAposCentimos": 700000,
  "criadoEm": "2026-10-01T09:30:00Z"
}
```

O destino é indicado pelo **número da conta** (que o cliente conhece); a origem, pelo seu id.

## Formato de erro (igual em toda a API)

```json
{
  "codigo": "SALDO_INSUFICIENTE",
  "mensagem": "O saldo da conta é insuficiente para esta operação.",
  "detalhes": null,
  "idPedido": "req-4f2a…"
}
```

O `idPedido` liga o erro ao registo de auditoria e aos logs.

## Catálogo de erros

| Código | HTTP | Quando |
|---|---|---|
| `PEDIDO_INVALIDO` | 400 | Validação de entrada falhou (`detalhes` lista os campos). |
| `CHAVE_IDEMPOTENCIA_EM_FALTA` | 400 | Transferência sem o cabeçalho. |
| `NAO_AUTENTICADO` | 401 | Token ausente, inválido ou expirado. |
| `CREDENCIAIS_INVALIDAS` | 401 | Login falhado (mensagem genérica, sem dizer se foi o utilizador ou a palavra-passe). |
| `SEM_PERMISSAO` | 403 | Perfil sem acesso à rota. |
| `RECURSO_NAO_ENCONTRADO` | 404 | Não existe **ou** não pertence a quem pede. |
| `CHAVE_IDEMPOTENCIA_REUTILIZADA` | 409 | Mesma chave com corpo diferente. |
| `SALDO_INSUFICIENTE` | 422 | RF-07, RF-08. |
| `CONTA_BLOQUEADA` | 422 | I-5. |
| `LIMITE_DIARIO_EXCEDIDO` | 422 | I-8. |
| `TRANSFERENCIA_MESMA_CONTA` | 422 | I-4. |
| `LIMITE_DE_PEDIDOS` | 429 | Limitação de pedidos excedida (RNF-10). |

## Aviso de implementação

O driver do PostgreSQL para Node devolve os `bigint` como **texto**. Converter de forma consistente ao ler e ao escrever, num único sítio (um *transformer* do TypeORM), e não espalhar conversões pelo código.

## Decisões

| # | Decisão | Razão |
|---|---|---|
| 1 | Nomes em português nas rotas e campos. | Coerência com a linguagem do domínio e com os documentos. |
| 2 | Dinheiro em cêntimos inteiros também na API. | Evita ambiguidade e conversões. |
| 3 | Paginação por cursor. | Com *offset*, movimentos novos entre pedidos causam linhas repetidas ou saltadas, e páginas fundas ficam lentas; o cursor apoia-se no índice do extracto. |
| 4 | RF-15 e seed de operadores e auditores. | Lacuna descoberta ao desenhar a API. |
