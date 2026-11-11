---
data: 2026-11-10
ticket: PF-93502
ticket_url: https://me.pro.mercadoe.rocks/pf-93502
mr:
  - https://gitlab.miisy.me/partner-ecosystem/integration-hub/-/merge_requests/309
projeto: Partner Portal (iPaaS)
tipo: bug
camada: [back]
---

# Integration Hub pede `taxClassification` nos itens de pré-pedido e pedido

## Resumo
O Hub passa a pedir `items.taxClassification` no pré-pedido e no pedido, pra o NCM/NBM aparecer na coluna INPUT (ME) do mapeamento. Fecha, junto com a [PF-93503](../2026-11-10_PF-93503_gateway-taxclassification-itens/README.md), o pedido da Moura Dubeux.

## Problema
A Moura Dubeux não conseguia mapear o NCM/NBM no de/para do configurador de integração no fluxo de pré-pedido. O Hub só pede ao GraphQL Gateway os grupos listados no manifest de tipos de documento, e o `taxClassification` não estava lá.

## Por que
Atender a Moura Dubeux (origem [PF-93453](https://me.pro.mercadoe.rocks/pf-93453)).

## Como
- **`document-types.manifest.json`:** `items.taxClassification` nos `relationshipPaths` de `pre-order` e de `order`.
- **`taxClassification` em vez de `productIdentificationCode`:** o segundo está marcado como obsoleto na API pública e não traz os códigos da reforma tributária (NBS, CST-CBSIBS, cClassTribCBSIBS, CST-IS, cClassTribIS).
- **Sem migration nova:** o `SeedOutboundDocumentTypesMigration` faz upsert por documento a cada deploy.
- **Ordem de deploy indiferente:** enquanto o Gateway não expõe o campo, o caminho é ignorado (já coberto em `GraphQlGatewayQueryBuilderTests`).
- Suíte completa verde (Domain, Application, Infrastructure, WebApi unit e integration).

## Onde
integration-hub: `document-types.manifest.json`

## Resultado / impacto
O NCM/NBM do item aparece na coluna INPUT (ME) do mapeamento do pré-pedido e do pedido, já no campo que traz os códigos da reforma tributária. No pedido, o valor só vem preenchido quando a API REST de pedido do ambiente devolver `taxClassification`; em STG ainda não devolve.

## Imagens
Sem prints.
