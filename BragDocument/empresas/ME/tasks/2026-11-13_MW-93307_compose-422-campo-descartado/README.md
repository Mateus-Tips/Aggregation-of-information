---
data: 2026-11-13
ticket: MW-93307
ticket_url: https://me.pro.mercadoe.rocks/pd-93307
mr:
  - https://gitlab.miisy.me/partner-ecosystem/integration-hub/-/merge_requests/301
projeto: Partner Portal (iPaaS)
tipo: bug
camada: [back]
---

# Compose responde 422 em vez de descartar campo mapeado

## Resumo
O compose resolvia conflito entre campos descartando um deles e devolvia só um warning, então o mapping era gravado sem o que o usuário ligou. Agora ele responde 422 com a lista de todos os campos afetados, e nada é gravado com perda.

## Problema
No conflito entre campos, o `POST /v1/mappings/compose` descartava ou ignorava um campo mapeado e respondia 200 com warning. O mapping era salvo sem esse campo, e o usuário não ficava sabendo.

## Por que
Salvar o mapping sem um campo que o usuário ligou é perda de dado silenciosa. É melhor recusar e apontar o que corrigir.

## Como
- **`ComposeWarningCodes.Blocking`:** reúne os códigos que bloqueiam: `leaf_conflict`, `structural_conflict`, `iterator_path_mismatch`, `parent_source_mismatch` e `field_dropped`.
- **`ComposeMappingHandler`:** se algum deles aparecer, lança `MappingConflictException` com todos os conflitos e registra o compose como falha na métrica.
- **`MappingConflictExceptionHandler`:** devolve 422 (ProblemDetails) com `conflicts: [{ target, reason, discardedExpression }]`, mais `target` (o primeiro campo) e `message`, no mesmo formato do 422 de parse.
- **Sem bloqueio:** `shape_normalized` e `index_list_source` continuam em `warnings` com 200. O `/preview` não muda e segue devolvendo tudo como warning.
- **`MappingConflictException`:** recusa lista vazia de conflitos.
- **Testes:** um conflito, vários conflitos na mesma resposta, só warning informativo com 200, métrica de falha e formato do 422.

## Onde
integration-hub: `ComposeMappingHandler`, `ComposeWarningCodes`, `MappingConflictException`, `MappingConflictExceptionHandler`

## Resultado / impacto
O Hub não grava mais mapping com perda de campo. Integração já gravada com um desses conflitos vai pedir correção do campo no próximo salvar, o que é esperado.

Depende do portal: hoje o BFF repassa só o `error` do 422 e descarta `target` e `conflicts`. Sem o MR do portal da MW-93308 (!539), o usuário fica com o toast genérico de falha ao salvar, sem saber qual campo corrigir. O portal vai usar o `conflicts[]` pra marcar os campos inline.

## Imagens
Sem prints.
