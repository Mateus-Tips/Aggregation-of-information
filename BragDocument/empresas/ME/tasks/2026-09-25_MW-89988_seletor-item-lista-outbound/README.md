---
data: 2026-09-25
ticket: MW-89988
ticket_url: https://meagil.atlassian.net/browse/MW-89988
mr:
  - https://gitlab.miisy.me/partner-ecosystem/integration-hub/-/merge_requests/260
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/488
projeto: Partner Portal (iPaaS)
tipo: bug
camada: [front, back]
---

# Seletor de item da lista no mapeamento outbound

## Resumo
No outbound, ligar um campo de lista num campo único do ERP mandava a lista inteira. Agora o canvas pergunta qual item usar e o Hub preserva o filtro.

## Problema
Ligar `businessOrganizations[].code` no `CompanyCode` do SAP mandava `"[\"111111\",\"131313\"]"`, sem aviso no canvas. Mesmo quando a IA gerava o filtro certo, o Hub descartava o predicado ao re-serializar o JSONata (`lista[campo='X'].code` virava `lista.code`).

## Por que
Campo escalar do ERP recebendo lista quebra a integração. Bug reportado em trunk com o pre-pedido 24788895.

## Como
- **Hub:** `JsonataSourceWriter` passa a serializar os predicados de cada passo do path (filtro por campo e por índice) e aceita `$filter`, `$lookup` e lambda no meio do path (antes dava 422). O classificador trata path com predicado como `expression`, pra não perder o filtro ao reabrir. Prompt outbound orienta a IA a filtrar por campo identificador, nunca por posição.
- **Front:** o canvas detecta ligação lista → escalar e abre um popover. O popover lista só campos com valor único em todos os itens (lido do sample) e mostra prévia. Sem campo identificador, usa o primeiro item com aviso. Sem escolha, o passo de mapeamento bloqueia.
- Extra: badge do wizard passa de Draft pra Needs review sem recarregar (o front esperava o status no PATCH, que o Hub nunca devolveu).

## Onde
- integration-hub: `JsonataSourceWriter`, `MappingAstService`, `MappingKindClassifier`, `mapping-core-outbound.md`
- partner-portal: `MappingCanvas`, `MappingListItemSelectorPopover`, helpers de transform/catalog, ADR-054

## Resultado / impacto
Campo único do ERP recebe um valor só. A escolha persiste ao salvar e reabrir. Validado ponta a ponta com dado real de trunk.

## Aprendizado
O bug parecia de front, mas era também de back: o serializador do AST jogava fora o predicado. Testar o round-trip (compose → decompose) pegou isso.

## Imagens
- [Seletor com dado real](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/0f27234559d094d3a113ed0a15558f6c/popover.png)
- [Mapeamento salvo e reaberto](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/98ea200d42eba8980774dfac1e65720a/05-after-reload.png)
