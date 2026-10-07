---
data: 2026-09-29
ticket: MW-89767
ticket_url: https://meagil.atlassian.net/browse/MW-89767
mr:
  - https://gitlab.miisy.me/partner-ecosystem/integration-hub/-/merge_requests/262  # MW-90125
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/491
projeto: Partner Portal (iPaaS)
tipo: feature
camada: [front, back]
---

# Instâncias fixas de BORGs e Attributes no mapeamento inbound

## Resumo
No inbound, agora dá pra montar várias BORGs/Attributes (`[0]`, `[1]`...) a partir de campos soltos do ERP. Antes só a primeira chegava no ME.

## Problema
O ERP manda entidades como campos soltos (`CompanyCode`, `PurchasingOrganization`), mas o ME espera uma lista com um objeto por entidade em `businessOrganizations`/`attributes`. O canvas só mostrava um molde da lista e o Hub só gerava iteração ou array de 1 objeto, então a segunda BORG nunca chegava.

## Por que
Integrações inbound reais precisam mandar mais de uma BORG (empresa, org de compras etc.) por documento.

## Como
- **Hub:** compose aceita targets indexados (`businessOrganizations[1].code`) e monta array literal com N objetos. Validações (422) pra lista fora da allowlist, mistura de indexado e não indexado, lacuna de índice. Decompose devolve as instâncias. A IA sugere instâncias como pendentes (confiança máx. 0.6).
- **Front:** botão "+ Adicionar instância" no lado ME. Cada `[n]` aceita ligação ou valor fixo, com remoção confirmada e renumeração. Ligar lista do ERP numa instância escolhe o item por posição, com badge "item N". Checagem de tipo vale também nessas ligações. Instâncias derivadas dos targets, sem campo novo no contrato.
- Teste novo de i18n que falha quando chave usada no `src` falta em algum locale.

## Onde
- integration-hub: `ListInstanceTarget`, `MappingFragmentBuilder`, `MappingComposer`, `GenerateMappingHandler`, `mapping-core.md`
- partner-portal: canvas do lado ME, `meCatalog`, ADR-057

## Resultado / impacto
ME recebe um objeto por instância. Fluxo publicado e reaberto em somente leitura preserva as instâncias.

## Aprendizado
Guardar o estado no catálogo (molde + quantidade) evitou mudar o contrato com o Hub.

## Imagens
- [Botão adicionar instância](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/ba5da355dc18562445af3ae5e5a09587/1-botao-adicionar-instancia.png)
- [Instâncias e preview com 2 BORGs](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/aaee2400a40d98db626a74a082f732f0/2-instancias-e-preview.png)
- [Seletor por posição](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/94ac515b1f3c6be657466419985fb866/3-seletor-por-posicao.png)
- [Badge item N](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/fdfd35210d82920b7702bba14c708c76/4-badge-item-junto-do-campo.png)
- [Confirmação de remoção](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/274c4c3f6a86ea24f644d14451732557/5-confirmar-remocao.png)
- [Tipo incompatível](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/e85eac8512463bdb2b2530fdfd90e79e/6-tipo-incompativel.png)
- [Somente leitura reaberto](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/b6e149df381872d42b9179ce8cac5600/7-somente-leitura-reaberto.png)
