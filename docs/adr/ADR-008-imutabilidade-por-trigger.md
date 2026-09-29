# ADR-008: Imutabilidade de movimentos e auditoria por trigger

- **Estado:** Aceite
- **Data:** 2026-09-29

## Contexto
Movimentos e registos de auditoria nunca podem ser alterados nem apagados (I-7), mesmo que exista um bug no código.

## Decisão
Um *trigger* na base de dados rejeita qualquer `UPDATE` ou `DELETE` em `movimentos` e `registos_auditoria`, criado numa migração.

## Alternativas consideradas
Retirar permissões de escrita ao utilizador da aplicação: mais robusto contra alguém que desactive o *trigger*, mas exige dois utilizadores de base de dados e mais configuração.

## Consequências
- (+) Simples de montar com migrações; a defesa vive na BD, fora do alcance de bugs da aplicação.
- (−) Um administrador da BD pode desactivar o *trigger*. Registado como risco residual no documento 7.
