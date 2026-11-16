---
data: 2026-11-16
ticket: PD-93608
ticket_url: https://me.pro.mercadoe.rocks/pd-93608
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/544
projeto: Partner Portal (iPaaS)
tipo: tech-debt
camada: [front, bff]
---

# Regras de lint pra `console` no BFF e `data-cy` fixo em `v-for`

## Resumo
Analisei os pontos de review dos 40 MRs mais recentes do partner-portal (!487 a !542), escolhi dois problemas que se repetem e passei a barrá-los com ESLint. Também corrigi o que as regras acusaram.

## Problema
Dois pontos voltavam com frequência no review:
- `console` no código do servidor (BFF);
- `data-cy` fixo em elemento com `v-for`, que se repete em cada item da lista e deixa o seletor ambíguo.

## Por que
O que se repete no review e dá pra checar automaticamente deve ser barrado pelo lint, e não depender de alguém lembrar.

## Como
- **`no-console`:** ligado em `src/api`, `src/services` e `src/middleware`. A única ocorrência (`integrations-document-types.js`) passou a usar o logger com escopo.
- **`vue/no-restricted-syntax`:** acusa `data-cy` fixo em elemento com `v-for`. O `eslint-plugin-vue` virou dependência direta, na mesma faixa que o `yarn.lock` já resolvia pelo `eslint-plugin-nuxt`, sem mudar o lockfile.
- **`data-cy` com o id do item:**
  - os 5 componentes que a regra acusou: `IntegrationWizardStepHeader`, `IntegrationWizardSteps`, `DocumentPayloadFeedback`, `ExecutionStepMetadataPanel` e `ExecutionTimelineEventList`;
  - os botões de adicionar e remover instância do `MappingMeTreeNode`, apontados no !491.
- **Testes:**
  - 6 specs unitários ajustados pros seletores novos;
  - as duas regras foram conferidas com arquivos que violam cada uma;
  - `yarn lint:fix` limpo e `yarn test` com 298 suítes e 4495 testes passando.
  - A suíte E2E não foi rodada.

## Onde
partner-portal: config do ESLint, `src/api` (`integrations-document-types.js`), 5 componentes do assistente e do monitoramento, `MappingMeTreeNode`, 6 specs

## Resultado / impacto
Os dois problemas passam a ser barrados no lint, antes do review. Nada muda na tela. A documentação da análise está no !543.

Limites e o que ficou de fora:
- A regra de `data-cy` só olha o elemento que tem o `v-for`. Um `data-cy` fixo dentro de um componente renderizado em lista por outro, como o `MappingMeTreeNode`, continua dependendo de review.
- O `eslint-plugin-vuejs-accessibility` fica pra outra tarefa.

## Imagens
Sem prints (sem mudança visual).
