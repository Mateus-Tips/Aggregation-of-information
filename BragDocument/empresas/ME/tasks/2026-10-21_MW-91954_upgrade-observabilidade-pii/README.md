---
data: 2026-10-21
ticket: MW-91954
ticket_url: https://meagil.atlassian.net/browse/MW-91954
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/158
projeto: Login Hipster
tipo: observabilidade
camada: [front, bff]
---

# Upgrade do módulo de observabilidade e telemetria sem dado pessoal no login

## Resumo
Subi o módulo de observabilidade do login de `^0.2.3` pra `^0.6.0` (SDK 0.18.0). Com isso a telemetria para de levar email, username e tokens.

## Problema
O login mandava email e username no `setUser`. As URLs dos spans e eventos (client e servidor) iam com query string (`token`, `guid`), e o username aparecia no path de `GET /login/v1/auth/<username>`.

## Por que
Tirar dado pessoal e token da telemetria. A busca de trace por usuário passa a ser pelo `user.id`, como foi alinhado com o tech lead.

## Como
- **`package.json` / `yarn.lock`:** módulo `^0.6.0` e SDK `@mercadoeletronico/me-frontend-observability` 0.18.0. `setUser` vai só com o `id` e as URLs saem sem query string.
- **`nuxt.config`:** `urlRedactions` no bloco `observability`, que troca `/v1/auth/<username>` por `/v1/auth/:username`. As outras rotas de auth não carregam dado no path e ficaram iguais.
- Conferi os changelogs do módulo (0.3.0 a 0.6.0) e do SDK (0.16.0 a 0.18.0): nenhum breaking change pro login.

## Onde
me-login-hipster: `package.json`, `yarn.lock`, `nuxt.config.js`

## Resultado / impacto
Telemetria do login sem email, username nem tokens, com trace por usuário buscado pelo `user.id`. O scrape do Prometheus no `/metrics` também deixou de gerar trace.

## Imagens
Sem prints.
