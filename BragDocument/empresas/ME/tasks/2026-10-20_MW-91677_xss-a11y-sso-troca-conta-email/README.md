---
data: 2026-10-20
ticket: MW-91677
ticket_url: https://meagil.atlassian.net/browse/MW-91677
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/157
projeto: Login Hipster
tipo: bug
camada: [front]
---

# XSS e acessibilidade no SSO, troca de conta e código por email

## Resumo
Corrigi duas XSS no login (`/email` e `/redirect`) e os problemas de acessibilidade e UX que os testes E2E da MW-91271 acharam nos fluxos de SSO, troca de conta e código por email.

## Problema
- Parâmetros de i18n renderizados com `v-html` sem escape: um `username` com HTML rodava na página `/email`.
- `/login/redirect?dest=javascript:...` executava JS na origem do login.
- Troca de conta não funcionava só pelo teclado, tinha elemento interativo aninhado, fonte de 12px e scroll interno; "Continuar nesta conta" voltava pro Keycloak.
- SSO sem alias quebrava, erros dos forms não eram anunciados pelo leitor de tela e o form de código não tinha como trocar o email nem autofill.

## Por que
Segurança da tela de entrada de todo o produto e acessibilidade dos fluxos de acesso, apontadas pela suíte E2E.

## Como
- **Segurança (`me-i18n.js`):** `escapeParameterHtml: true` no VueI18n; todo `v-html` com parâmetro passa a mostrar o valor como texto.
- **Segurança (`redirect.vue`):** `dest` só da mesma origem; `javascript:` ou outro domínio cai no `/mercado.asp`. Todos os chamadores já passam por ele, então nenhum fluxo muda.
- **Troca de conta (`AccountList.vue`, `switch.vue`):** `<button>` nativo cobrindo o card (Enter/Espaço, foco visível), `aria-current` na conta atual, sem 12px e sem scroll interno. "Continuar nesta conta" vai pro `dest` ou `/mercado.asp` via `/redirect`.
- **SSO (`sso/_alias.vue`, `index.vue`):** com alias segue pro provedor (alias encodado no `kc_idp_hint`); sem alias volta pro login com aviso no idioma certo.
- **Feedback (`Feedback.vue`):** `role="alert"`, erros de todos os forms anunciados.
- **Código por email (`EmailCodeForm.vue`, `email.vue`):** erros limpos a cada envio, timer de reenvio só depois de sucesso, botão "Alterar e-mail" e `autocomplete="one-time-code"`/`inputmode="numeric"` via diretiva local (o `me-text-input` do toolkit não aceita esses atributos).
- **Locales:** 3 chaves novas nos 6 idiomas.
- **Testes:** unitários e E2E de XSS, teclado na troca de conta, `/redirect` com `javascript:` e outra origem, SSO sem alias e alterar email.

## Onde
me-login-hipster: `me-i18n.js`, `pages/redirect.vue`, `AccountList.vue`, `pages/switch.vue`, `pages/sso/_alias.vue`, `pages/index.vue`, `Feedback.vue`, `EmailCodeForm.vue`, `pages/email.vue`, locales

## Resultado / impacto
Duas XSS fechadas na tela de login, troca de conta usável só pelo teclado e no mobile, erros anunciados pelo leitor de tela em todos os forms. Os achados vieram da suíte E2E (MW-91271), retorno direto do investimento em testes.

## Imagens
Troca de conta com foco de teclado no segundo card

![01-switch-keyboard-focus](https://gitlab.miisy.me/identity/meweb/me-login-hipster/uploads/06402df5c26916082d72865f11c81a45/01-switch-keyboard-focus.png)

Troca de conta no mobile (16px, sem scroll interno)

![02-switch-mobile](https://gitlab.miisy.me/identity/meweb/me-login-hipster/uploads/6bbe46b87463ea435916f4096e6e5626/02-switch-mobile.png)

Payload HTML no `username` aparece como texto

![03-email-xss-as-text](https://gitlab.miisy.me/identity/meweb/me-login-hipster/uploads/ae2c0a9d059c12e1e0f8132db80a5266/03-email-xss-as-text.png)

Código inválido com orientação + "Alterar e-mail"

![04-email-invalid-code-change-email](https://gitlab.miisy.me/identity/meweb/me-login-hipster/uploads/d637134b543b4dbb90aaf42ea8bc2da5/04-email-invalid-code-change-email.png)

`/sso/` sem alias volta pro login com aviso

![05-sso-missing-alias](https://gitlab.miisy.me/identity/meweb/me-login-hipster/uploads/b74e7080a8576b7e894adea44aa7581e/05-sso-missing-alias.png)
