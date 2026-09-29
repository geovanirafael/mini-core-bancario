# 3. Modelo de domínio

> **Estado:** aceite · **Data:** 29 de Setembro de 2026

## Como se modela um domínio

1. **Substantivos dos requisitos → candidatas a entidades.**
2. **Verbos → operações.**
3. **Entidade ou valor?** Uma *entidade* tem identidade própria (a conta 0012 é a mesma ao longo do tempo, mesmo que o saldo mude). Um *valor* é definido só pelo conteúdo (100 Kz é 100 Kz).
4. **Invariantes:** afirmações que têm de ser verdadeiras *sempre*, seja qual for o caminho que o código percorra. Viram restrições na base de dados e testes.
5. **Ciclos de vida:** estados e transições permitidas.

## Entidades

| Entidade | O que representa | Campos principais |
|---|---|---|
| **Utilizador** | Quem se autentica | id, identificador, hash da palavra-passe, perfil (cliente/operador/auditor), estado |
| **Cliente** | Pessoa titular de contas (1:1 com um Utilizador de perfil cliente) | id, nome, documento de identificação fictício |
| **Conta** | Onde está o dinheiro | id, número único, cliente, saldo, estado (activa/bloqueada), limite diário |
| **Operação** | Um acto de negócio concluído: depósito, levantamento ou transferência | id, tipo, valor, autor, chave de idempotência, data |
| **Movimento** | O efeito de uma operação numa conta (imutável) | id, operação, conta, sentido (crédito/débito), valor, saldo após, data |
| **Registo de auditoria** | Rasto de quem fez o quê | id, autor, acção, alvo, resultado, detalhes, data |

**Valor:** *Dinheiro* (montante em cêntimos de kwanza).

## Relações

- Um Cliente tem **várias** Contas; uma Conta pertence a **um** Cliente.
- Uma Conta tem **muitos** Movimentos.
- Uma Operação gera **1 ou 2** Movimentos: depósito e levantamento geram um; transferência gera dois (débito na origem, crédito no destino).
- Um Utilizador é autor de muitas Operações e de muitos Registos de auditoria.

## Ciclo de vida da conta

`activa` ⇄ `bloqueada`. Só há um estado em cada momento e só o operador muda entre eles (RF-13). Encerrar contas fica fora do MVP.

## Invariantes

| ID | O que nunca pode deixar de ser verdade |
|---|---|
| I-1 | O saldo de uma conta nunca é negativo. |
| I-2 | Todo o valor de uma operação é positivo. |
| I-3 | Uma transferência gera exactamente dois movimentos, de valor igual e sentidos opostos. |
| I-4 | A conta de origem e a de destino de uma transferência são diferentes. |
| I-5 | Uma conta bloqueada não recebe nem emite movimentos. |
| I-6 | O saldo de uma conta é igual à soma dos seus créditos menos a soma dos seus débitos. |
| I-7 | Movimentos e registos de auditoria nunca são alterados nem apagados; só se acrescentam. |
| I-8 | A soma das transferências enviadas por uma conta num dia não ultrapassa o seu limite diário. |

**Propriedade de conservação:** numa transferência, o dinheiro não se cria nem se destrói. Só depósitos e levantamentos alteram o total de saldos do sistema. Esta propriedade verifica-se no fim de qualquer teste.

## Decisões

| # | Decisão | Alternativa | Razão |
|---|---|---|---|
| 1 | **Guardar o saldo na conta**, protegido pela invariante I-6 (verificada por testes e por uma consulta de reconciliação). | Calcular sempre a partir dos movimentos. | Consulta instantânea e uma linha única onde bloquear para a concorrência; é o que os sistemas reais fazem. |
| 2 | **Dinheiro em cêntimos inteiros** (1 500,50 Kz = 150050). | Vírgula flutuante ou decimal. | Sem arredondamentos surpresa (RNF-02). |
| 3 | **Operações falhadas só na auditoria.** Uma operação existe apenas se foi concluída. | Guardar operações com estado "falhada". | Mantém o histórico financeiro limpo; o rasto das falhas fica completo na auditoria. |
| 4 | **Utilizador e Cliente separados.** | Fundir numa só entidade. | Operador e auditor são utilizadores mas não titulares de contas. |
