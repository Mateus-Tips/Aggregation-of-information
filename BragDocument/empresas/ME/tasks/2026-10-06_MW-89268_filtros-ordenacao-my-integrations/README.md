---
data: 2026-10-06
ticket: MW-89268
ticket_url: https://meagil.atlassian.net/browse/MW-89268
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/503
projeto: Partner Portal (iPaaS)
tipo: feature
camada: [front]
---

# Filtros e ordenação em My Integrations

## Resumo
My Integrations ganhou filtro por status e por direção (Inbound/Outbound) e ordenação por nome ou última alteração (padrão: mais recente primeiro).

## Problema
A lista de integrações não tinha filtro nem ordenação, então era difícil achar uma integração específica.

## Por que
Pedido da task (produto). Os filtros resetam a cada acesso, como alinhado com Produto.

## Como
- Toolbar com 3 selects (Status, Direção, Ordenar por) e estado vazio próprio quando o filtro não retorna nada.
- Filtro e ordenação em memória (`myIntegrationsListing`), sem mutar a lista. A listagem não é paginada (até 100 fluxos), então vale pra lista inteira.
- O card passa a carregar `direction` e `lastModifiedAt` (`updatedAt` ou `createdAt`).
- Bug achado no caminho: `SidebarListPageLayout` checava o slot `toolbar` num computed sobre `$slots` (não reativo), então toolbar condicional nunca aparecia. Passou a checar no template via `$scopedSlots`.

## Onde
partner-portal: página `my-integrations`, `myIntegrationsListing.js`, `integrationCardMapper.js`, `SidebarListPageLayout.vue`

## Resultado / impacto
O usuário acha integrações por status e direção e vê as mais recentes primeiro.

Pendências: a ordenação por última alteração depende do MW-90812 (Hub expor `createdAt`/`updatedAt`). Filtro por tipo de documento ficou pra depois.

## Aprendizado
`$slots` não é reativo no Vue 2: computed sobre ele não atualiza.

## Imagens
- [Padrão: mais recente primeiro](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/417e2683cae9819467745854c838c726/01-default.png)
- [Opções de ordenação](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/d033806792b96cd3c73db76ed0b0ca0b/02-sort-open.png)
- [Ordenado por nome](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/941c1af3c45aa5f2dc2739f56a7794d1/03-sorted-by-name.png)
- [Filtro de status](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/554a970a2c5bd3f39d196a54674a967e/04-status-open.png)
- [Validated + Outbound](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/71f5618f4a3c8965ca750a226f481917/06-validated-outbound.png)
- [Sem resultados](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/c8354e097715870b8be644814c9e9303/07-no-results.png)
