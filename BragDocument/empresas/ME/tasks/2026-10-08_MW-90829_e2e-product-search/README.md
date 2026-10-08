---
data: 2026-10-08
ticket: MW-90829
ticket_url: https://meagil.atlassian.net/browse/MW-90829
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/506
projeto: Partner Portal (iPaaS)
tipo: testes
camada: [front]
---

# E2E do Product Search

## Resumo
O Product Search (wizard de catálogo) ganhou E2E com o mesmo ciclo dos outros tipos. Com isso os 3 tipos de integração (Inbound, Outbound, Product Search) têm E2E de ponta a ponta.

## Problema
Product Search era o único tipo de integração sem E2E. O wizard é diferente (duas operações, getAll e getById, cada uma com aba própria) e depende de um provedor externo (Grainger) instável.

## Por que
Fecha a série MW-90827 / MW-90828: regressão automática dos 3 fluxos de integração.

## Como
- **`catalogWizardPage`:** page object do wizard de catálogo, estendendo o do wizard genérico. Reaproveita a espera do canvas com retry da IA e a confirmação dos vínculos pendentes.
- **Fluxo:** escolhe o connector Grainger e o fornecedor, configura getAll e getById, preenche os payloads, aguarda os mapeamentos da IA, roda o teste real nas duas operações, confere o Review e publica. Depois reabre em somente leitura, suspende, exclui e confere que o card sumiu (`reopenReadOnly` e `suspendAndDelete` reaproveitados).
- **Provedor externo:** teste real com 1 retry. Se o Hub ainda apontar `failedAtStep: "API_REQUEST"` (provedor fora), o teste vira `skip` com o `correlationId`. Qualquer outro erro falha. Connector inexistente também vira `skip`.
- **Payloads:** exemplos de resposta em `tests/e2e/payloads/`, aplicados com `setValue` do CodeMirror (digitar alguns KB de JSON era lento demais).

## Onde
partner-portal: `tests/e2e/` (`catalogWizardPage.js`, `integration-create.spec.js`, payloads, fixtures, README)

## Resultado / impacto
Suíte com 8 testes passando e 1 skip em 4.6 min no trunk. O skip foi do Product Search, porque o getById do Grainger estava respondendo erro. Parâmetros, payloads, mapeamentos e o teste do getAll passaram. Sem mudança em código de produção.

## Aprendizado
Com dependência externa instável, separar "falha do provedor" (skip com `correlationId` pra rastrear) de "falha nossa" (vermelho) mantém o E2E confiável sem esconder bug.

## Imagens
Sem prints (mudança só em testes).
