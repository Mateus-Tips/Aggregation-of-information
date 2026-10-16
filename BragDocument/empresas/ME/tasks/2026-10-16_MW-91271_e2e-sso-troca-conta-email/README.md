---
data: 2026-10-16
ticket: MW-91271
ticket_url: https://meagil.atlassian.net/browse/MW-91271
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/153
projeto: Login Hipster
tipo: testes
camada: [front]
---

# E2E de SSO, troca de conta e login por código de email

## Resumo
E2E com Playwright pros fluxos de acesso além de usuário e senha: SSO por alias, SSO pelo domínio do email, troca de conta e login por código de email, cada um com caminho feliz e cenários de erro.

## Problema
O E2E do login (MW-91270) só cobria usuário, senha e idioma. SSO, troca de conta e código por email não tinham teste de ponta a ponta.

## Por que
Sequência da MW-91270 na meta de cobertura do Login Hipster. Usa os helpers de mock criados lá.

## Como
- **`support/api.js`:** mocks novos pra usuário logado, opções de troca de conta, código de email, SMS, criação de usuário e páginas legadas do `/do/MEIdSSO.mvc`, pra os redirects não saírem do ambiente local.
- **`sso.e2e.js`:** `/sso/:alias` redireciona pro IdP com `kc_idp_hint`. SSO por domínio (`isSsoActive`): cadastro sem etapa de senha via `POST /login/v1/users`, com e sem telefone, SMS inválido, domínio sem SSO indo pra `/password/new` e erro na criação. Cenários vieram da suíte do QA (`sso.cy.js`).
- **`switch.e2e.js`:** lista de contas com a atual marcada, seleção com e sem "lembrar", "Continuar nesta conta", "Sair da sessão" e expirado quando falta `guid` ou o `GetSwitchOptions` falha.
- **`email.e2e.js`:** envio do código (body e token conferidos), erro e expirado no envio, código válido indo pro LoginMEId ou pro cadastro, código inválido.

## Onde
me-login-hipster: `tests/e2e/support/api.js`, `tests/e2e/sso.e2e.js`, `tests/e2e/switch.e2e.js`, `tests/e2e/email.e2e.js`

## Resultado / impacto
Suíte E2E com 25/25 passando em 3 execuções seguidas. Os pontos de UX, acessibilidade e uma XSS achados nesses fluxos viraram a [MW-91677](https://meagil.atlassian.net/browse/MW-91677).

## Aprendizado
Reaproveitar os cenários da suíte do QA (Cypress) evitou reinventar casos de SSO por domínio.

## Imagens
Sem prints.
