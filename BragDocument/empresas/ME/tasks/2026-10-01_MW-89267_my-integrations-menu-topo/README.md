---
data: 2026-10-01
ticket: MW-89267
ticket_url: https://meagil.atlassian.net/browse/MW-89267
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/496
projeto: Partner Portal (iPaaS)
tipo: melhoria-ux
camada: [front]
---

# Menu "Integration" abre My Integrations

## Resumo
Clicar em "Integration" no menu do topo agora leva pra My Integrations, não Templates.

## Problema
O usuário caía em Templates, mas o uso mais comum é ver as próprias integrações.

## Por que
Pedido da task (produto).

## Como
O item do menu aponta pra `/integrations/my-integrations`. Na sidebar, My Integrations virou o primeiro item. Specs do menu e dos layouts ajustadas.

## Onde
partner-portal: `topNavItems`, layouts `default` e `integrations`

## Resultado / impacto
Um clique a menos pro fluxo principal. Breadcrumb e wizard já apontavam pra lá, então a navegação ficou consistente.

## Imagens
- [My Integrations como destino](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/7c54945df0bbb1eb12cdedb4259ddb8e/01-my-integrations.png)
