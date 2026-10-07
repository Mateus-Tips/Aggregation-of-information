---
data: 2026-10-01
ticket: MW-90301
ticket_url: https://meagil.atlassian.net/browse/MW-90301
mr:
  - https://gitlab.miisy.me/partner-ecosystem/integration-hub/-/merge_requests/265  # MW-90300
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/494
projeto: Partner Portal (iPaaS)
tipo: feature
camada: [front, back]
---

# Instâncias fixas em listas aninhadas (subnível)

## Resumo
Extensão das instâncias fixas pro subnível: `items › attributes`, `locations › businessOrganizations` etc. Cada item do ERP vira um item do ME com o mesmo conjunto de objetos e os seus próprios valores.

## Problema
O MW-89767 só cobria listas na raiz. Atributos por item (ex.: Cor e Material de cada item) não tinham como ser montados a partir de campos soltos do ERP.

## Por que
Follow-up direto do MW-89767, que deixou listas aninhadas fora de escopo.

## Como
- **Hub:** allowlist ganha as listas aninhadas. Índice só na lista de dentro (`items.attributes[1].value`), e o compose monta o array literal dentro da projeção do item pai. A fonte do pai vem dos campos comuns, dos `erpPaths` ou do envelope da expression. Warning `parent_source_mismatch` quando instâncias apontam listas diferentes pro pai.
- **Front:** o "+" aparece nas listas aninhadas (o nó da árvore passou a repassar o evento). Campo do próprio item é ligação direta; lista dentro do item segue a regra por posição. A reabertura passou a preservar valor fixo dentro de lista iterada (antes sumia, vale também pra listas comuns).

## Onde
- integration-hub: `ListInstanceTarget`, `MappingFragmentBuilder`, `MappingComposer`, `GenerateMappingHandler`, `mapping-core.md`
- partner-portal: `integrationListInstances.js`, `mappingListInstances.js`, `MappingMeTreeNode.vue`, `integrationHubMappingAdapter.js`, ADR-057

## Resultado / impacto
Salvar e reabrir preserva instâncias, valores fixos e preview. Regressão E2E de Generic Inbound e Outbound sem divergência. Varredura dos mappings salvos em trunk: nenhum afetado pela mudança de leitura.

## Aprendizado
Antes de mudar como algo é lido no decompose, varrer os dados já salvos pra ver se algo existente muda de forma.

## Imagens
- [Canvas com instâncias aninhadas](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/4acff69a6d0cdbef4c9f49f3f86a8fe1/01-canvas-nested-instances.png)
- [Seletor por posição](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/da52961e7104ded83a8d78ecadb4ba35/02-position-selector.png)
- [Badge por posição](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/941c37ab7204358891f85db8bc7c3493/03-canvas-position-badge.png)
- [Reaberto](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/b223e009449a578fa9ca256877ec8e3e/05-reopened.png)
- [locations › businessOrganizations](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/305bfe1222ec3c9acd7da885a5407583/06-locations-canvas.png)
