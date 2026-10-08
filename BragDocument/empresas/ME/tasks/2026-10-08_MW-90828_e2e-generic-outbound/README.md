---
data: 2026-10-08
ticket: MW-90828
ticket_url: https://meagil.atlassian.net/browse/MW-90828
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/505
projeto: Partner Portal (iPaaS)
tipo: testes
camada: [front]
---

# E2E do Generic Outbound

## Resumo
O Generic Outbound ganhou E2E com o mesmo ciclo do Inbound: criar, mapear com IA, testar, publicar, reabrir em somente leitura, suspender e excluir. O mapeamento no E2E também ficou mais estável no trunk.

## Problema
Só o Inbound tinha E2E de ciclo completo (MW-90827). No trunk, o mapeamento da IA às vezes falhava (502) e os vínculos pendentes travavam o Next, deixando o teste instável.

## Por que
Continuação do MW-90827: cobrir os 3 tipos de integração (Inbound, Outbound, Product Search) em E2E.

## Como
- **Reuso:** `selectTemplateAndCreateFlow(templateId)` serve pra inbound e outbound. Reabrir em somente leitura e suspender/excluir viraram `reopenReadOnly` e `suspendAndDelete`, usados pelos dois specs.
- **Parâmetros:** `fillParameters` aceita `identifier`. Se o pedido fixo do trunk não existir, o teste vira `skip` em vez de falhar (problema de ambiente, não de código).
- **Mapeamento:** confirma os vínculos pendentes da IA quando aparecem e, se a análise falhar, clica em "Tentar novamente" uma vez antes de falhar.
- **Outbound:** teste novo com o pedido `39587058`, cobrindo criar, publicar, somente leitura, suspender e excluir.
- `data-cy` no botão "Confirmar pendentes" e README do E2E atualizado.

## Onde
partner-portal: `tests/e2e/` (`integration-create.spec.js`, page objects, README), `MappingCanvas.vue`

## Resultado / impacto
7 testes passando no trunk em 2.8 min, cobrindo Inbound e Outbound de ponta a ponta. Sem mudança visual (só `data-cy`). Product Search vem no !506 (MW-90829).

## Aprendizado
Separar falha de ambiente (dado que não existe no trunk) de falha de código: `skip` em vez de vermelho evita ruído no CI.

## Imagens
Sem prints (mudança só em testes).
