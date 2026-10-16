---
data: 2026-10-16
ticket: MW-91266
ticket_url: https://meagil.atlassian.net/browse/MW-91266
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/150
projeto: Login Hipster
tipo: testes
camada: [front]
---

# Testes unitários do cadastro e correção do SMS sem validação

## Resumo
Testes do `RegistrationForm` e da página de cadastro, que estavam com 0% de cobertura. Os testes acharam um bug: um erro genérico na validação do SMS deixava o cadastro avançar.

## Problema
Formulário e página de cadastro sem nenhum teste. No `onValidateToken`, erro de rede ou 5xx chamava `done()` depois do erro, e como o `done` do form chama `proceed()`, o usuário seguia pra criar conta ou definir senha sem o SMS validado.

## Por que
Parte da meta de 80% de cobertura do Login Hipster (sequência da MW-91265).

## Como
- **`RegistrationForm.spec.js`:** campos obrigatórios, tamanho máximo e padrão de nome, telefone inválido, envio direto sem telefone, envio de SMS, validação do código, reenvio com timer de 30s e botão de voltar.
- **`registration.spec.js`:** `asyncData` (redirect pra `/expired` sem token/email, parse de `sso`), `onSendToken`, `onValidateToken` e `onNextStep` com sucesso, 401, 400 e erro genérico, fluxo sso e não-sso.
- **Correção em `registration.vue`:** no erro genérico do `onValidateToken`, só mostra o erro e continua no passo do código.

## Onde
me-login-hipster: `RegistrationForm.vue`, `pages/registration.vue` e specs

## Resultado / impacto
`RegistrationForm.vue` com 92,13% de linhas e 88,13% de branches; `registration.vue` com 100% de linhas e 95,45% de branches. Fechou um caminho em que dava pra criar conta sem validar o SMS.

## Aprendizado
Testar os caminhos de erro, não só o de sucesso, foi o que achou o bug.

## Imagens
Sem prints.
