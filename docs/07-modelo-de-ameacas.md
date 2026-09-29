# 7. Segurança e modelo de ameaças

> **Estado:** aceite · **Data:** 29 de Setembro de 2026

Este é um modelo de ameaças de projecto de portfólio, **não uma auditoria de conformidade**. Um banco real tem requisitos regulatórios e de certificação que este projecto não pretende cumprir.

## Método

1. **Activos:** o que tem valor.
2. **Atacantes:** quem quer prejudicar ou abusar.
3. **Fronteiras de confiança:** onde os dados passam de um sítio menos fiável para um mais fiável.
4. **Ameaças**, enumeradas com **STRIDE**: *Spoofing* (fingir ser outro), *Tampering* (alterar dados), *Repudiation* (negar ter feito algo), *Information disclosure* (fuga de dados), *Denial of service* (indisponibilidade), *Elevation of privilege* (fazer o que não se pode).
5. **Controlos:** para cada ameaça, onde ela é travada. Princípio: **defesa em profundidade**, sem depender de uma só barreira.

## Contexto

- **Activos:** saldos e movimentos (o dinheiro), credenciais e tokens, registos de auditoria, dados pessoais dos clientes.
- **Atacantes:** cliente malicioso (autenticado, quer ver ou mexer no que não é seu); atacante externo anónimo; operador desonesto (interno, com privilégios); quem obtém um token roubado.
- **Fronteiras de confiança:** Internet → API (tudo o que entra é não fiável e é validado); API → base de dados (só a API fala com ela).

## Ameaças e controlos

| Ameaça | STRIDE | Controlo |
|---|---|---|
| Força bruta ou reutilização de credenciais no login | S | Limitação de pedidos mais rigorosa no login, atraso progressivo, hash lento (argon2/bcrypt), mensagem genérica (`CREDENCIAIS_INVALIDAS`) |
| Token forjado ou roubado | S | Assinatura com segredo forte, algoritmo fixado no servidor (rejeitar `none`), expiração curta |
| Cliente acede à conta de outro (*BOLA/IDOR*) | I, E | Verificação de propriedade **em cada acesso, no servidor**; 404 em vez de 403; UUIDs não adivinháveis |
| Cliente chama rotas de operador | E | Guards por perfil; **protegido por omissão** (RNF-11) |
| *Mass assignment*: o cliente envia `perfil: "operador"` ou `saldoCentimos` no corpo | T, E | DTOs com lista branca; campos desconhecidos rejeitados (RNF-12); nunca receber entidades directamente |
| Valores negativos, decimais ou gigantes | T | Validação (inteiro, > 0, máximo definido) **e** `CHECK` na BD |
| Repetição de um pedido (*replay*) ou duplo clique | T | Chave de idempotência (RF-14) |
| Dois pedidos simultâneos gastam o mesmo saldo | T | Bloqueio pessimista de linhas (documento 5) |
| Injecção SQL | T | ORM com parâmetros; **regra: nunca concatenar texto em consultas cruas**, sobretudo no `FOR UPDATE` |
| Adulteração ou remoção de registos de auditoria | T, R | *Trigger* de imutabilidade; auditor só de leitura |
| Erros ou logs que expõem dados internos | I | Filtro global de erros; nunca registar palavras-passe nem tokens (RNF-13) |
| Enumeração de contas ou utilizadores | I | 404 uniforme; login com mensagem genérica |
| Pedidos massivos ou extractos gigantes | D | Limitação de pedidos (RNF-10); `limite` máximo por página; tempos-limite |
| Segredos no repositório Git | I | `.env` fora do Git, com um `.env.example` sem valores reais (RNF-13) |
| Dependências vulneráveis | T, E | `package-lock` versionado; `npm audit` regular |
| Transporte inseguro ou CORS aberto | I | HTTPS obrigatório em produção; CORS restrito à origem do frontend; cabeçalhos de segurança (`helmet`) |
| Palavra-passe inicial fraca ou conhecida do operador | S | Gerada aleatoriamente pelo servidor e mostrada uma só vez (RF-15); troca no primeiro acesso (RF-16) |

## Riscos residuais (aceites e documentados)

- **Operador desonesto:** pode registar depósitos fictícios. A auditoria deixa rasto e há separação de funções (o operador não transfere, o auditor não movimenta), mas não há aprovação em dois níveis. Evolução futura.
- **Administrador da BD com acesso total:** pode desactivar o *trigger*. A protecção real seria registos encadeados por *hash* ou armazenamento só de escrita.
- **Sem revogação imediata de tokens:** como não há *refresh tokens*, um token roubado vale até expirar. É o custo da simplicidade decidida no ADR-003.
- **Sem autenticação de dois factores.**

## Requisitos gerados por este exercício

RNF-10 (limitação de pedidos), RNF-11 (protegido por omissão), RNF-12 (lista branca nos DTOs), RNF-13 (segredos fora do Git e logs limpos) e RF-16 (troca de palavra-passe). Ver o documento 2.

## Decisões

| # | Decisão | Razão |
|---|---|---|
| 1 | Travar a força bruta com **limitação de pedidos e atraso progressivo**, sem bloqueio permanente de contas. | O bloqueio após N falhas cria um ataque novo: bloquear contas alheias de propósito. |
| 2 | Palavra-passe inicial **gerada pelo servidor** e mostrada uma só vez, com troca no primeiro acesso. | O operador não deve conhecer a palavra-passe do cliente. Se o RF-16 for cortado, passa a risco residual. |
| 3 | **Aceitar e documentar os riscos residuais** acima. | Dizer com clareza o que fica de fora é sinal de maturidade. |
