---
data: 2026-10-07
ticket: MW-90827
ticket_url: https://meagil.atlassian.net/browse/MW-90827
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/504
projeto: Partner Portal (iPaaS)
tipo: testes
camada: [front]
---

# E2E do ciclo de vida das integrações + smokes

## Resumo
O E2E do Generic Inbound agora cobre o ciclo inteiro (publicar, reabrir, editar ativa, republicar, suspender, excluir), além do cancelamento da criação e de smokes de My Integrations e Connectors. Também corrigi o spec do Inbound, que estava quebrado na develop.

## Problema
O E2E parava na publicação, e o spec do Inbound estava quebrado: não passava pelo modal de ativação e pegava o id do flow da chamada errada (qualquer `/flows/*`).

## Por que
Regressão do fluxo principal do produto rodando de ponta a ponta, sem depender de teste manual.

## Como
- **Publicação:** marca o checkbox de secret salvo e clica em Continuar no modal de ativação (`data-cy` novos, com teste unitário do botão).
- **Id do flow:** lido do POST `/api/integrations/flows/{templateId}` com caminho exato. A espera pela URL do wizard ficou depois do id salvo, então o `afterEach` limpa o flow mesmo se o redirect falhar.
- **Inbound:** depois de publicar, reabre em somente leitura, edita com ela ativa, troca o nome, refaz payload, mapeamento e teste, republica, suspende, exclui e confere que o card sumiu.
- **Cancelar criação:** usa "Cancelar criação" no header e confere pela resposta do GET `/api/integrations/flows`, não pelo DOM, pra não passar antes do grid renderizar.
- **Smokes:** My Integrations carrega, form de Connectors abre (criar e editar). Qualquer escrita em connectors é barrada com `page.route` e faz o teste falhar.
- README do E2E com escopo, layout e convenções.

## Onde
partner-portal: `tests/e2e/` (specs, page objects, fixtures, README), `IntegrationActivatedSuccess.vue`, `ConnectorFormShell.vue`

## Resultado / impacto
6 testes passando no trunk em 1.3 min. Sem mudança visual (só `data-cy`). Outbound (MW-90828) e Product Search (MW-90829) vêm em MRs separados.

## Aprendizado
- Interceptar por caminho exato, não por glob, pra não pegar a resposta errada.
- Conferir "sumiu da lista" pela resposta da API, não pelo DOM, pra evitar falso positivo.
- Smoke seguro em ambiente compartilhado: bloquear escrita com `page.route`.

## Imagens
Sem prints (mudança só em testes).
