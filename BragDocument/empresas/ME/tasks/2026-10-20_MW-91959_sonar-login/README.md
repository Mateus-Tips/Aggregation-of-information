---
data: 2026-10-20
ticket: MW-91959
ticket_url: https://meagil.atlassian.net/browse/MW-91959
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/159
projeto: Login Hipster
tipo: tech-debt
camada: [front]
---

# Zerar issues do SonarQube no Login Hipster

## Resumo
Zerei as issues abertas no SonarQube do me-login-hipster: 6 bugs, os code smells e o security hotspot, sem mudar o fluxo de login.

## Problema
O projeto tinha bugs, code smells e um hotspot de segurança abertos no Sonar, a maioria de acessibilidade e código morto.

## Por que
Deixar o projeto limpo no Sonar depois da leva de testes e observabilidade, pra que issues novas apareçam de verdade.

## Como
- **Imagens:** logos com `alt="Mercado Eletrônico"`; ilustrações decorativas com `alt=""` (o alerta de "image" redundante era falso positivo, o Sonar leu o nome da chave i18n).
- **Locales:** chaves `login.invitation.*.image` removidas dos 6 idiomas.
- **LoginForm:** `autocomplete="current-passowrd"` corrigido pra `current-password`; "Esqueceu sua senha?" de `<label>` clicável pra `<button type="button">`, agora com foco pelo teclado.
- **Forms:** `submit()` deixa de retornar `false` (já tem `@submit.prevent`); regex `[0-9]` vira `\d`.
- **PasswordResetForm:** try/catch trocado por `errors?.[...]`, com teste pro 400 sem `errors`.
- **Loading:** `div role="status"` vira `<output>`.
- **Health check:** `X-Powered-By: Express` desligado no `/healthz` (hotspot S5689), com teste.

## Onde
me-login-hipster: layouts, `LoginForm.vue`, `PasswordForm.vue`, `RegistrationForm.vue`, `PasswordResetForm.vue`, componente de loading, health check, locales

## Resultado / impacto
Sonar do login zerado (bugs, code smells e hotspot). As issues restantes do `invitation/index.vue` e do `AccountList.vue` já tinham sido fechadas na MW-91274 e na MW-91677.

## Imagens
Sem prints.
