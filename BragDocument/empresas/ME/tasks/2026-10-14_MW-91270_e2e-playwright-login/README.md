---
data: 2026-10-14
ticket: MW-91270
ticket_url: https://meagil.atlassian.net/browse/MW-91270
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/146
projeto: Login Hipster
tipo: testes
camada: [front]
---

# Setup de E2E com Playwright no login

## Resumo
Primeira entrega dos testes E2E do Login Hipster com Playwright: `yarn test:e2e` sobe o app sozinho e roda login com sucesso, senha errada e troca de idioma com a API mockada, sem depender de ambiente.

## Problema
O login não tinha teste E2E. Testar o fluxo de ponta a ponta dependia de ambiente e de usuário real no trunk.

## Por que
Parte da meta de cobertura do Login Hipster. Deixa a base e os mocks prontos pras próximas subtarefas (SSO, troca de conta, cadastro e senha).

## Como
- **Setup (`playwright.config.js`, `package.json`):** `@playwright/test` e script `test:e2e`. O `webServer` sobe o `yarn dev` numa porta só do E2E (4190) e espera o bundle ficar pronto (`/login/_nuxt/app.js`), não a tela de loading. Força `LOCAL_USERNAME` vazio pro app não logar no trunk.
- **Mocks (`tests/e2e/support/api.js`):** helpers com `page.route` pra sessão, consulta de usuário e signin. Chamada pra `/login/v1` sem mock é abortada, então mock esquecido falha na hora em vez de bater no trunk.
- **Specs (`tests/e2e/*.e2e.js`):** login com sucesso indo pra `/login/email`, senha errada mostrando erro, idioma do navegador (`en-US`), troca pra inglês e persistência depois do reload. Seletores `data-cy`.
- Sufixo `.e2e.js` pro Jest não pegar os specs, sem mudar o `jest.config.js`.

## Onde
me-login-hipster: `playwright.config.js`, `package.json`, `tests/e2e/`, README, `.gitignore`, `.prettierignore`

## Resultado / impacto
O time roda o E2E do login local, sem ambiente. Validado com 3 execuções seguidas sem falha, e `yarn test` segue igual.

## Aprendizado
Abortar qualquer chamada sem mock deixa o E2E determinístico e evita bater em ambiente real sem perceber.

## Imagens
Sem prints.
