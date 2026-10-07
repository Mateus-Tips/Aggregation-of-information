---
data: 2026-10-01
ticket: MW-90480
ticket_url: https://meagil.atlassian.net/browse/MW-90480
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/141
projeto: Login Hipster
tipo: melhoria-ux
camada: [front]
---

# Melhorias na tela de troca de conta

## Resumo
A tela de troca de conta ficou mais clara: lista com scroll, conta logada destacada com badge "Conta atual" e link "Continuar nesta conta", e um único botão "Sair da sessão".

## Problema
A lista de contas crescia sem limite, a conta logada não se destacava e não tinha um jeito direto de voltar pra onde o usuário estava.

## Por que
Usuário com várias contas vinculadas precisa identificar rápido em qual está e voltar sem trocar.

## Como
- **AccountList.vue:** altura máxima `max(300px, 45vh)` com scroll interno, hover e transição nos cards. Conta logada com fundo azul claro (`--me-primary-7`), badge `me-badge` "Conta atual" e link "Continuar nesta conta →" que emite `continue` sem disparar a seleção do card.
- **switch.vue:** "Acessar outra conta" trocado por "Sair da sessão" (mesmo logout). O `continue` chama `window.history.back()`, então o usuário volta pra página onde estava. No primeiro login não tem conta logada, então o link não aparece.
- Chaves novas nos 6 locales, testes do badge, do link e do `history.back()`.

## Onde
me-login-hipster: `AccountList.vue`, `pages/switch.vue`, locales, `jest.config.js`

## Resultado / impacto
Usuário vê na hora qual conta está ativa e volta pra onde estava com um clique.

Limitação: aberta direto numa aba nova (sem histórico), o "Continuar nesta conta" não faz nada.

## Imagens
- [Tela de troca de conta](https://gitlab.miisy.me/identity/meweb/me-login-hipster/uploads/e5505dc25a16f859eb018a68d76dcb08/switch-evidence-5.png)
