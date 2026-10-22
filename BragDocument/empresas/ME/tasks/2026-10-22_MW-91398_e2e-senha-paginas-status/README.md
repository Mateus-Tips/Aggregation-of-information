---
data: 2026-10-22
ticket: MW-91398
ticket_url: https://meagil.atlassian.net/browse/MW-91398
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/160
projeto: Login Hipster
tipo: testes
camada: [front]
---

# E2E dos fluxos de senha e páginas de status do Login Hipster

## Resumo
É a última entrega da história de E2E do login ([MW-91263](https://meagil.atlassian.net/browse/MW-91263)). Cobri com Playwright o reset de senha, a criação de nova senha e as páginas de sessão expirada e não autorizado, tanto no caminho feliz quanto nos cenários de erro.

## Problema
Os fluxos de senha e as páginas de status não tinham E2E. Os testes de nova senha estavam soltos em `migration.e2e.js`.

## Por que
Fechar a cobertura E2E do login, completando o que as subtarefas anteriores (MW-91270 a MW-91272) começaram.

## Como
- **`password.e2e.js`:** reset de senha com sucesso (200 vai pra `/password/success`), usuário desconhecido (404 vai pra `/password/error`), usuário sem migração (422 vai pra recuperação legada), erro 500, campo vazio e cancelar.
- **`password.e2e.js`:** página de sucesso (timer de reenvio, reenviar, voltar) e página de erro.
- **`password.e2e.js`:** criação de nova senha com senha fraca, confirmação diferente, erro genérico, token rejeitado (401) e acesso sem token indo pra página de expirado. Os testes que estavam em `migration.e2e.js` vieram pra cá.
- **`status.e2e.js`:** páginas de expirado e não autorizado, com texto e botão de voltar mantendo o `dest`.
- **`login.e2e.js`:** link "esqueci minha senha".
- **Helpers:** `createPassword` em `support/forms.js`, `mockResetPassword` novo e `mockLegacyPages` cobrindo `Login.mvc` em `support/api.js`.

## Onde
me-login-hipster: `e2e/password.e2e.js`, `e2e/status.e2e.js`, `e2e/login.e2e.js`, `e2e/migration.e2e.js`, `e2e/support/forms.js`, `e2e/support/api.js`

## Resultado / impacto
A suíte E2E do login passou de 38 pra 65 testes, e passou em 3 execuções seguidas. Contra o build de produção roda em cerca de 2 min; contra o `yarn dev` leva 8 a 9 min e tem timeouts esporádicos, por causa do bundle de vendors servido pelo dev server.

## Imagens
Sem prints.
