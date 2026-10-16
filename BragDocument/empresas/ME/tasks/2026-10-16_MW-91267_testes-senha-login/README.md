---
data: 2026-10-16
ticket: MW-91267
ticket_url: https://meagil.atlassian.net/browse/MW-91267
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/151
projeto: Login Hipster
tipo: testes
camada: [front]
---

# Testes unitários de criação e reset de senha

## Resumo
Testes dos forms de criação e reset de senha e das páginas `password/new` e `password/reset`, que estavam com 0% de cobertura. Sem mudança em código de produção.

## Problema
Fluxo de senha sem nenhum teste unitário.

## Por que
Parte da meta de 80% de cobertura do Login Hipster (sequência da MW-91265).

## Como
- **`PasswordForm`:** regras de senha (tamanho, maiúscula e minúscula, número, caractere especial), bloqueio do submit quando falta uma regra ou a confirmação não bate, emissão da senha e retorno `done` / `errors` do pai.
- **`PasswordResetForm`:** usuário pela query, chamada do reset, navegação pra sucesso, erro (404) e recuperação legada, toast por status (400, 422, 500, sem resposta) e botão de voltar mantendo o `dest`.
- **`pages/password/new`:** `asyncData` com e sem token, criação do usuário com os dados da store e o token, redirect pro ME ID, 401 pra expirado e erro genérico.
- **`pages/password/reset`:** título, seletor de idioma e form.

## Onde
me-login-hipster: `PasswordForm.vue`, `PasswordResetForm.vue`, `pages/password/new.vue`, `pages/password/reset.vue`

## Resultado / impacto
100% de statements, linhas e funções nos 4 arquivos. Branches entre 83% e 95,8%; o `reset.vue` mostra 50% porque o único branch dele é gerado pelo vue2-jest.

## Aprendizado
O branch que falta em todo SFC vem do código gerado pelo vue2-jest (`typeof exports.default === 'function' ? ...`), não do componente. Vale saber antes de correr atrás de 100% de branches.

## Imagens
Sem prints.
