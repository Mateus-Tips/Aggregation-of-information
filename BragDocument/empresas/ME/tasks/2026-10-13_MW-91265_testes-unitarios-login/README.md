---
data: 2026-10-13
ticket: MW-91265
ticket_url: https://meagil.atlassian.net/browse/MW-91265
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/145
projeto: Login Hipster
tipo: testes
camada: [front]
---

# Testes unitários do login e cobertura real do projeto

## Resumo
Primeira entrega dos testes unitários do login: a cobertura passou a medir o `src/` inteiro (o número real era 18%, não 77,9%) e subiu pra 30,7% com testes do `LoginForm` e da página de login.

## Problema
A cobertura só contava arquivos que já tinham teste, então mostrava 77,9% quando o real era 18%. O fluxo de login não tinha teste unitário.

## Por que
Primeira subtarefa da meta de 80% de cobertura no Login Hipster. Sem medir o projeto todo, o número escondia o que faltava testar.

## Como
- **Jest (`jest.config.js`):** `collectCoverageFrom` em `src/**/*.{js,vue}`, sem `nuxt.config.js` e `plugins`.
- **`LoginForm`:** 16 testes cobrindo os dois passos (usuário e senha), validação, `localStorage`, voltar e reset da validação entre os passos.
- **Página de login:** 18 testes cobrindo `asyncData`, `checkUser` (ME ID, usuário bloqueado, login legado com e sem proxy), `signIn` e o parâmetro de idioma.
- Testes validados quebrando o código de propósito (passo 2, fluxo bloqueado, `return` depois do redirect, reset da validação): todos falharam e voltaram a passar.
- O `coverageThreshold` de 80% fica pra última subtarefa (MW-91269), pra não quebrar a CI no meio.

## Onde
me-login-hipster: `jest.config.js`, testes do `LoginForm` e da página de login. Sem mudança em `src/`.

## Resultado / impacto
Cobertura honesta (18% → 30,7% global), com `LoginForm` e página de login acima de 80% de linhas e branches cada.

## Aprendizado
Cobertura medida só sobre arquivos testados engana. Configurar `collectCoverageFrom` pro projeto todo antes de definir meta.

## Imagens
Sem prints.
