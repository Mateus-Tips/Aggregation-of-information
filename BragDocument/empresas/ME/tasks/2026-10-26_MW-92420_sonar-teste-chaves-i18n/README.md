---
data: 2026-10-26
ticket: MW-92420
ticket_url: https://meagil.atlassian.net/browse/MW-92420
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/162
projeto: Login Hipster
tipo: tech-debt
camada: [front, bff]
---

# Últimos code smells do Sonar e teste de chaves i18n no Login Hipster

## Resumo
Zerei os 2 code smells que ainda estavam abertos no Sonar do develop. Também trouxe pro login o teste que o partner-portal já tem, que barra chave i18n usada no código sem valor nos locales.

## Problema
Sobraram 2 code smells no Sonar depois da MW-91959. Além disso, nada impedia uma chave i18n nova sem tradução de chegar em produção e aparecer crua na tela.

## Por que
Manter o Sonar do login zerado e pegar chave sem tradução na CI em vez de em produção.

## Como
- **BFF `/metrics`:** quando a coleta do prom-client falha, o erro é logado com `console.error` antes do 500 (S2486).
- **LoginForm:** `autocomplete` do usuário virou `:autocomplete="'username'"`. O S6840 é falso positivo do Sonar 25.7 com componente custom (`me-text-input`), já corrigido upstream na SONARHTML-253. O browser continua recebendo `autocomplete="username"`.
- **Teste i18n (`tests/locales/usedKeys.spec.js`):** procura em `src` as chamadas `$t` / `$i18n.t` / `$i18n.tc` com chave literal e falha se alguma não tiver valor em algum locale. Também exige que todo locale tenha as mesmas chaves do `en-US`, o que cobre as chaves montadas em runtime.
- **Testes:** `dest` malformado no `sameOriginDest`, log do erro no `/metrics` e, no E2E de login, o `autocomplete` real dos campos de usuário e senha.

## Onde
me-login-hipster: `src/api/metrics.js`, `LoginForm.vue`, `tests/locales/usedKeys.spec.js`, `e2e/login.e2e.js`

## Resultado / impacto
Sonar do develop zerado. Chave i18n sem tradução agora quebra a CI, no mesmo padrão do partner-portal. Ficou de fora só a chave montada em runtime que falte em todos os locales, inclusive no `en-US`.

## Imagens
Sem prints.
