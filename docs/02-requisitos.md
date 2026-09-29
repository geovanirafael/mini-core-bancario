# 2. Requisitos

> **Estado:** aceite · **Data:** 29 de Setembro de 2026 (inclui os requisitos acrescentados nas etapas 6 e 7)

## Como ler este documento

- **RF** = requisito funcional (o que o sistema faz). **RNF** = requisito não funcional (como se comporta).
- Cada requisito tem um **ID** para haver rastreabilidade: testes, endpoints e commits apontam para ele.
- Prioridade **MoSCoW**: *Must* (sem isto o MVP não existe), *Should* (importante, mas o MVP sobrevive sem), *Could* (só se sobrar tempo).
- Um requisito só é válido se for **testável**.

## Requisitos funcionais

| ID | Requisito | Prioridade |
|---|---|---|
| RF-01 | Um utilizador autentica-se com identificador e palavra-passe e recebe uma credencial com prazo de validade. | Must |
| RF-02 | Existem três perfis (cliente, operador, auditor); cada operação só é permitida aos perfis autorizados, verificado no servidor. | Must |
| RF-03 | O operador abre uma conta associada a um cliente; a conta tem número único, saldo inicial zero e estado "activa". | Must |
| RF-04 | Um cliente pode ter várias contas. | Must |
| RF-05 | O cliente consulta o saldo apenas das suas próprias contas. | Must |
| RF-06 | O operador regista um depósito (valor > 0), que aumenta o saldo e gera um movimento. | Must |
| RF-07 | O operador regista um levantamento (valor > 0); é rejeitado se o saldo for insuficiente. | Must |
| RF-08 | O cliente transfere valor de uma conta sua para outra conta activa e diferente; a operação é atómica. | Must |
| RF-09 | Cada conta tem um limite diário de transferências, configurável; ultrapassá-lo rejeita a operação. | Should |
| RF-10 | O cliente consulta o extracto das suas contas, por período, paginado, por ordem cronológica. | Must |
| RF-11 | Toda a operação sensível (login, registo de cliente, abertura de conta, depósito, levantamento, transferência, e também as falhas) gera um registo de auditoria com quem, quando, o quê e resultado. | Must |
| RF-12 | O auditor consulta os registos de auditoria; ninguém os pode alterar ou apagar. | Must |
| RF-13 | O operador bloqueia ou desbloqueia uma conta; uma conta bloqueada não movimenta. | Should |
| RF-14 | Cada pedido de transferência leva uma chave de idempotência, para que repetir o mesmo pedido não duplique a operação. | Should |
| RF-15 | O operador regista um cliente (cria o utilizador de perfil cliente e a respectiva ficha). A palavra-passe inicial é gerada pelo servidor, forte e aleatória, e mostrada uma única vez ao operador. Operadores e auditores são criados por script de seed. | Must |
| RF-16 | O utilizador altera a sua própria palavra-passe (esperado no primeiro acesso de um cliente). | Should |

## Requisitos não funcionais

| ID | Requisito | Prioridade |
|---|---|---|
| RNF-01 | Palavras-passe guardadas só como hash (bcrypt ou argon2). | Must |
| RNF-02 | Valores monetários nunca em vírgula flutuante; guardados em inteiros na menor unidade (cêntimos). | Must |
| RNF-03 | Transferências, depósitos e levantamentos executam em transacção da base de dados (tudo ou nada). | Must |
| RNF-04 | O saldo nunca fica negativo, garantido também por uma restrição na própria base de dados, não só no código. | Must |
| RNF-05 | Duas operações simultâneas na mesma conta nunca gastam mais do que o saldo disponível. | Must |
| RNF-06 | Todas as entradas são validadas; os erros seguem um formato consistente e não expõem detalhes internos. | Must |
| RNF-07 | A API é documentada em OpenAPI/Swagger. | Should |
| RNF-08 | As regras de negócio têm testes automatizados. | Must |
| RNF-09 | O extracto responde em tempo razoável (meta indicativa: menos de 500 ms com 10 000 movimentos). | Could |
| RNF-10 | Limitação de pedidos (rate limiting) em todas as rotas, mais estrita no login. | Must |
| RNF-11 | Rotas protegidas por omissão (deny-by-default); só as marcadas como públicas dispensam autenticação. | Must |
| RNF-12 | DTOs com lista branca de campos; campos desconhecidos são rejeitados. | Must |
| RNF-13 | Segredos em variáveis de ambiente, nunca no Git; sem palavras-passe nem tokens nos logs. | Must |

## Critérios de aceitação dos requisitos críticos

Formato **Dado / Quando / Então**.

- **RF-07:** *Dado* uma conta com 1 000 Kz, *quando* o operador levanta 1 000 Kz, *então* o saldo é 0 e existe um movimento de débito. *Dado* a mesma conta, *quando* tenta levantar 1 000,01 Kz, *então* recebe `SALDO_INSUFICIENTE` e o saldo mantém-se.
- **RF-08:** *Dado* uma conta A com 10 000 Kz e uma conta B activa, *quando* o dono de A transfere 3 000 Kz para B, *então* A fica com 7 000 Kz, B aumenta 3 000 Kz e existe um movimento em cada conta. *Dado* uma conta A com 1 000 Kz, *quando* o dono tenta transferir 3 000 Kz, *então* a operação é rejeitada e nenhum saldo muda.
- **RF-09:** *Dado* uma conta com limite diário de 5 000 Kz e 4 000 Kz já transferidos hoje, *quando* o cliente transfere 1 500 Kz, *então* recebe `LIMITE_DIARIO_EXCEDIDO`; com 1 000 Kz, a transferência passa.
- **RF-11:** *Dado* uma transferência rejeitada por saldo insuficiente, *então* não existe nenhuma operação nem movimento novos, **mas existe** um registo de auditoria com resultado "falha".
- **RF-14:** *Dado* o mesmo pedido enviado duas vezes com a mesma chave, *então* existe uma única operação e as duas respostas são iguais.

## Decisões

| # | Decisão | Razão |
|---|---|---|
| 1 | Um cliente pode ter várias contas (RF-04). | Modelo de dados mais realista (relação 1 para muitos). |
| 2 | Sem descoberto: o saldo nunca é negativo (RNF-04). | Descoberto traz juros e limites de crédito, já excluídos do âmbito. |
| 3 | Limite diário de transferência (RF-09) como *Should*, configurável por conta. | Enriquece as regras sem bloquear o MVP; o valor concreto define-se mais tarde. |
| 4 | RF-15 e o seed de operadores e auditores. | Lacuna descoberta ao desenhar a API: faltava dizer como nascem os utilizadores. |
| 5 | RF-16 e RNF-10 a RNF-13 acrescentados. | Resultam do modelo de ameaças (documento 7). |
