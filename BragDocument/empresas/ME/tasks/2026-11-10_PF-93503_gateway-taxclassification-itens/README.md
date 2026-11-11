---
data: 2026-11-10
ticket: PF-93503
ticket_url: https://me.pro.mercadoe.rocks/pf-93503
mr:
  - https://gitlab.miisy.me/meweb/api/graphql-gateway/-/merge_requests/9
projeto: Partner Portal (iPaaS)
tipo: bug
camada: [back]
---

# GraphQL Gateway expõe `taxClassification` nos itens de pré-pedido e pedido

## Resumo
A Moura Dubeux não conseguia mapear o NCM/NBM no de/para do configurador de integração no fluxo de pré-pedido. O Gateway descartava o campo; agora ele declara `taxClassification` nos itens de pré-pedido e de pedido.

## Problema
O input do outbound vem do GraphQL Gateway. A API REST de itens já devolvia `taxClassification`, mas o schema do Gateway não declarava o campo, então ele era descartado e não aparecia no mapeamento.

## Por que
Atender a Moura Dubeux (origem [PF-93453](https://me.pro.mercadoe.rocks/pf-93453)), que precisa do NCM/NBM no de/para do pré-pedido.

## Como
- **`src/schemas/preorder.graphql`:** campo `taxClassification: [PreOrderItemTaxClassification]` no `PreOrderItem`, com o tipo novo (`type` e `value`).
- **`src/schemas/order.graphql`:** mesmo campo no `OrderItem` (`[OrderItemTaxClassification]`).
- Sem resolver nem chamada REST nova: o dado já vem na resposta de `/items` que o Gateway consome.
- **Validação:**
  - check `syncing-graphql-contracts` (dry run): os dois campos saíram da lista de faltando;
  - `yarn eslint` e `yarn test` verdes, e o servidor sobe com o SDL mesclado;
  - consulta real em STG (org Moura), pré-pedido `15396175`: volta `[{ "type": "NCM/NBM", "value": "56031250" }]`.

## Onde
graphql-gateway: `src/schemas/preorder.graphql`, `src/schemas/order.graphql`

## Resultado / impacto
O NCM/NBM do item de pré-pedido passa a chegar pelo Gateway. O Integration Hub pede o campo na [PF-93502](../2026-11-10_PF-93502_hub-taxclassification-itens/README.md).

Pendências:
- No pedido, o campo volta `null` em STG porque a API REST de pedido ainda não devolve `taxClassification` nesse ambiente. Quando passar a devolver, o Gateway entrega sem outra mudança.
- O check também aponta `taxClassification` faltando em `contract` e `invoice`, que ficaram fora.

## Imagens
Sem prints.
