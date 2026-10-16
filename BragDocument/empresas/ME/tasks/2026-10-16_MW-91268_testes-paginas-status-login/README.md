---
data: 2026-10-16
ticket: MW-91268
ticket_url: https://meagil.atlassian.net/browse/MW-91268
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/152
projeto: Login Hipster
tipo: testes
camada: [front]
---

# Testes unitários das páginas de status, convite, SSO e email

## Resumo
Testes das páginas de status, convite, SSO, redirect e email do login, que estavam com 0% de cobertura. Todas ficaram com 100% de linhas, sem mudança em código de produção.

## Problema
Essas páginas não tinham teste. Email e SSO só tinham os testes de observabilidade.

## Por que
Parte da meta de 80% de cobertura do Login Hipster (sequência da MW-91265).

## Como
- **`password/success` e `password/error`:** username da query, contador de reenvio de 30s, reenvio indo pra `/password/reset` e voltar mantendo o `dest`.
- **`expired` e `unauthorized`:** título, texto, seletor de idioma e voltar mantendo o `dest`.
- **`invitation/index`:** troca do token, carga do convite, locale e os redirects (expirado, aceito por status ou `userId`, cadastro quando a regra cria usuário).
- **`invitation/accepted` e `expired`, `sso/_alias`, `redirect`:** renderização, alias ausente e navegação pro `dest`.
- **`email`:** `asyncData` com e sem token, envio do código (sucesso, 401, erro genérico), validação (ME ID, cadastro com token novo, 401, código inválido, erro genérico) e eventos do `EmailCodeForm`.

## Onde
me-login-hipster: `pages/password/`, `pages/expired.vue`, `pages/unauthorized.vue`, `pages/invitation/`, `pages/sso/_alias.vue`, `pages/redirect.vue`, `pages/email.vue`

## Resultado / impacto
100% de cobertura de linhas em todas as páginas do escopo.

## Imagens
Sem prints.
