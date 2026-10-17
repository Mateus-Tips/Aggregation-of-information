---
data: 2026-10-16
ticket: MW-91269
ticket_url: https://meagil.atlassian.net/browse/MW-91269
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/155
projeto: Login Hipster
tipo: testes
camada: [front]
---

# Testes da infraestrutura do app e gate de 80% de cobertura

## Resumo
Última entrega da história MW-91262: testes da infraestrutura do Login Hipster e gate de 80% de cobertura em linhas e branches na CI. A cobertura de linhas, que era 18% no início da história, fechou em 97,29%.

## Problema
Store, middleware, utils, layout, seletor de idioma e endpoint de health não tinham teste. Sem gate, nada impedia a cobertura de cair de novo.

## Por que
Fechar a meta de cobertura da história MW-91262 e garantir que ela não se perca.

## Como
- **`store/index`:** estado inicial, mutations, actions gravando no `localStorage` e no `sessionStorage`, getters com fallback pro storage.
- **`middleware/cookie-redirect`:** redirect pra `/mercado.asp` com sessão, verificação em cache sem sessão, 401 sem log e demais erros logados.
- **`utils/destUtil`:** `appendDestParam` e `buildRedirectUrl` com e sem `dest`.
- **`LocaleDropDown`:** locale salvo, idioma do navegador com fallback `pt-BR` e troca ao clicar.
- **`layouts/default`, `api/health`, `pages/switch`:** branding, resposta `healthy` e redirect pra conta escolhida com `dest` e `rememberme`.
- **`jest.config.js`:** `coverageThreshold` global de 80% em linhas e branches. O `yarn test --coverage` (mesmo comando do Jenkins) falha abaixo disso.
- Sem mudança em código de produção.

## Onde
me-login-hipster: `jest.config.js`, README e specs de store, middleware, utils, layout, `LocaleDropDown`, `api/health`, `pages/switch`

## Resultado / impacto
Cobertura global de linhas de 89,75% pra 97,29% e branches de 82,08% pra 88,59%. A CI agora barra MR que derrube a cobertura abaixo de 80%.

História MW-91262 completa: 18% → 97,29% de linhas, com unitários (MW-91265 a MW-91269) e E2E (MW-91270 a MW-91272).

## Aprendizado
Ativar o gate só na última subtarefa deixou entregar a história em partes sem quebrar a CI no meio.

## Imagens
Sem prints.
