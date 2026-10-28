---
data: 2026-10-20
ticket: MW-91984
ticket_url: https://meagil.atlassian.net/browse/MW-91984
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/519
projeto: Partner Portal (iPaaS)
tipo: feature
camada: [front, bff]
---

# Integration Hub liberado pra todos os tenants

## Resumo
Removi a allowlist de tenants do Integration Hub. Todo parceiro autenticado agora acessa Integrações, Execuções e Envio de Documentos.

## Problema
O acesso dependia de uma lista de IDs de tenant no código, checada no front (middleware) e no BFF (403 por tenant). Liberar um parceiro exigia deploy.

## Por que
Abrir o Integration Hub pra todos os parceiros, sem cadastro manual por tenant.

## Como
- **BFF (`src/api`):** remove o `requireIntegrationHubAccess` das rotas de integrations-* e document-submission; o token continua exigido.
- **Páginas:** remove o middleware `integration-hub-preview` de Integrações, Execuções e Envio de Documentos.
- **Menu do topo (`topNavItems`):** Integrações e Monitoramento pra todos; Tráfego e Log de eventos saem do topo e ficam só dentro de Monitoramento.
- **Sidebar de Monitoramento:** sempre lista Execuções, Tráfego, Eventos e Envio de Documentos.
- **Remoções:** `utils/integrations/integrationHubAccess.js` e `middleware/integration-hub-preview.js`.
- **Testes:** specs ajustados; saem os casos de bloqueio por tenant e o E2E de layout checa o item Integrações visível.

## Onde
partner-portal: `src/api`, páginas de Integrações, Execuções e Envio de Documentos, `topNavItems`, sidebar de Monitoramento

## Resultado / impacto
Integration Hub disponível pra todos os parceiros, sem deploy por tenant e com menos código (allowlist e middleware removidos).

## Imagens
Sem prints.
