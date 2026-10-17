---
data: 2026-10-16
ticket: MW-91272
ticket_url: https://meagil.atlassian.net/browse/MW-91272
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/154
projeto: Login Hipster
tipo: testes
camada: [front]
---

# E2E de migração, cadastro com SMS e convite

## Resumo
E2E com Playwright pras jornadas de cadastro do login: migração do usuário legado até a nova senha, registro com SMS e seletor de país, e convite (pendente com e sem SSO, aceito e expirado). A suíte ficou com 38 testes estáveis.

## Problema
As jornadas de cadastro não tinham E2E. Ao crescer a suíte, o primeiro carregamento passava de 28s e estourava o timeout de 30s.

## Por que
Última parte do E2E do Login Hipster (sequência da MW-91270 e MW-91271).

## Como
- **`migration.e2e.js`:** signin, `/email`, código por email, registro e `/password/new`, conferindo o `POST /login/v1/users` (nome, senha, token) e o redirect pro `LoginMEId`. Também migração `Blocked` e erros 500 e 401 na nova senha.
- **`registration.e2e.js`:** registro sem SSO escolhendo Portugal no seletor de país, com o código do país no `sms/send` e erro de formato quando a API recusa o telefone.
- **`invitation.e2e.js`:** troca do token (`exchange-token`), convite pendente abrindo o registro, com SSO criando o usuário sem senha, idioma do convite aplicado e páginas de aceito e expirado.
- **`support/api.js`:** mock do lookup de país por IP do vue-tel-input (`ip2c.org`). Sem ele, a resposta podia chegar depois do telefone digitado e trocar o país, quebrando o teste fora do Brasil ou sem rede.
- **`support/forms.js`:** passos repetidos (`signIn`, `confirmEmail`, `fillName`, `confirmSms`) viraram helper compartilhado.
- **`playwright.config.js`:** `workers: 4`. Com 8 workers cada um baixava o bundle de dev de 16MB do mesmo servidor e estourava o timeout; com 4 a suíte ficou estável e mais rápida (~1.3min).
- Cenários vieram da suíte do QA (`login.cy.js`, `invitation.cy.js`, `invitation-join-organization.cy.js`, `sso.cy.js`).

## Onde
me-login-hipster: `tests/e2e/` (`migration`, `registration`, `invitation`, `support/api.js`, `support/forms.js`), `playwright.config.js`

## Resultado / impacto
Suíte E2E do login com 38/38 passando em 3 execuções seguidas, cobrindo login, SSO, troca de conta, código por email, migração, cadastro e convite.

## Aprendizado
Mais workers nem sempre é mais rápido: com servidor de dev compartilhado, 4 workers foi mais estável e mais rápido que 8. Requisição externa não mockada (lookup de país) também deixa teste instável.

## Imagens
Sem prints.
