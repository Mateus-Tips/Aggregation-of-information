---
data: 2026-10-07
ticket: MW-89272
ticket_url: https://meagil.atlassian.net/browse/MW-89272
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/508
projeto: Partner Portal (iPaaS)
tipo: feature
camada: [front]
---

# KPIs do mapeamento viraram filtros clicáveis

## Resumo
Os KPIs do topo da tela de mapeamento (De-Para) viraram filtros: clicar num card mostra só os campos daquele indicador na coluna de destino, clicar de novo limpa.

## Problema
O pedido era filtrar só os campos obrigatórios. Os KPIs só mostravam números, e o card "obrigatórios sem conexão" não ajudava a achar os campos.

## Por que
Em mapeamentos grandes, achar o que falta mapear (obrigatórios, pendentes da IA) exigia rolar a lista inteira. No refinamento visual o escopo cresceu do toggle "somente obrigatórios" pra todos os KPIs clicáveis.

## Como
- **MappingSummaryBar:** cards viram botões com ícone de funil, sombra e hover com elevação. Card ativo fica azul e o funil vira "x". Cards com 0 ficam escondidos (menos conexões e o filtro ativo).
- **Obrigatórios:** card mostra o total de obrigatórios, com o número vermelho enquanto faltar algum mapear.
- **MappingCanvas:** filtra só a renderização da coluna de destino; conexões, valores fixos e preview não mudam. Se a validação ou o preview apontar erro num campo escondido, o filtro desliga sozinho. Filtro sem campos mostra mensagem com botão pra limpar.
- **`filterMeCatalog`:** filtra por conexões, IA, manuais, pendentes, valores fixos e obrigatórios, mantendo os grupos com algum campo visível.
- Acessibilidade: `role="group"` + `aria-label` e `aria-pressed` nos cards.
- Canvas de Connectors usa o mesmo componente e continua com cards estáticos.
- Testes unitários e E2E novo (`mapping-filters.spec.js`): filtra obrigatórios, preenche valor fixo com filtro ativo, limpa e confere que nada se perdeu.

## Onde
partner-portal: `MappingSummaryBar.vue`, `MappingCanvas.vue`, `integrationMappingCatalog.js`, locales, `tests/e2e/mapping-filters.spec.js`

## Resultado / impacto
Usuário vai direto nos campos que importam (obrigatórios, pendentes da IA) sem perder nada do que já mapeou. O filtro não é salvo, sempre abre desligado.

## Aprendizado
Filtrar só a renderização, sem mexer no estado do mapeamento, deixou o filtro seguro: nenhuma conexão ou valor some ao filtrar.

## Imagens
- [Sem filtro](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/da57e554132585a77f79e391cd7dafdf/01-sem-filtro.png)
- [Hover no card](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/7e91c116efa4058a79b5ab6a54f5cc49/02-hover-kpi.png)
- [Filtro Obrigatórios ativo](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/994242b44a93fcd4c02ef3e9db3dd497/03-filtro-obrigatorios.png)
- [Filtro IA ativo](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/5a6e1cab9384ac6d1c15609a459473cf/04-filtro-ia.png)
