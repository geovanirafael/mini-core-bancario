# ADR-003: JWT de acesso curto, sem refresh tokens

- **Estado:** Aceite
- **Data:** 2026-09-29

## Contexto
É preciso autenticação simples e segura para um MVP com prazo curto.

## Decisão
JWT de acesso de curta duração, assinado com segredo forte e algoritmo fixado no servidor. Sem *refresh tokens* no MVP: quando expira, o utilizador autentica-se de novo.

## Alternativas consideradas
Par de tokens (acesso + *refresh*) com rotação e revogação.

## Consequências
- (+) Menos peças e menor superfície de ataque.
- (−) Sem revogação imediata: um token roubado vale até expirar. Registado como risco residual no documento 7.
- Os *refresh tokens* ficam como evolução futura.
