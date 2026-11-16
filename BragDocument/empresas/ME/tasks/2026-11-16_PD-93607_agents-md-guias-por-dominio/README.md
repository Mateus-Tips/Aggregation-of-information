---
data: 2026-11-16
ticket: PD-93607
ticket_url: https://me.pro.mercadoe.rocks/pd-93607
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/543
projeto: Partner Portal (iPaaS)
tipo: tech-debt
camada: [front, bff]
---

# `AGENTS.md` enxuto e guias do agente por domínio

## Resumo
O `AGENTS.md` do partner-portal caiu de 314 pra 55 linhas. O resto foi pra guias em `docs/agents/`, que o agente lê só quando a tarefa é daquele domínio, com regras novas tiradas do review dos 40 MRs mais recentes.

## Problema
O `AGENTS.md` tinha 314 linhas e misturava regra que vale pra toda tarefa com regra de domínio, mapa de pastas e trechos de código.

## Por que
O agente carregava tudo em toda tarefa. E os pontos que mais voltavam no review não estavam escritos em lugar nenhum.

## Como
- **Fonte das regras novas:** análise dos pontos de review dos MRs !487 a !542. Só entrou o que apareceu em 2 ou mais MRs.
- **`AGENTS.md`:** descrição, comandos, regras de toda tarefa, formato de commit e índice dos guias. Ganhou a regra de apagar as sobras ao remover ou renomear algo e a lista "Check every path" (tipos de fluxo, modos, chamadores, BFF e front, estados, idiomas, docs), que cobre o achado mais comum: mudança que funciona no caminho testado e falha em outro.
- **Guias em `docs/agents/`:**
  - `ui.md`: Vue, componentes, ícones, SCSS e i18n. Novo: teclado em elemento clicável, comparar locale por idioma, mensagem de upstream fora do toast.
  - `testing.md`: Jest e Playwright. Novo: cenários que o review mais cobrou (lista vazia, tudo zerado, salvar e reabrir), `data-cy` com id em `v-for`, espera com poll no E2E e o que não precisa de teste.
  - `bff.md`: estrutura, segurança e logging. Novo: corpo da requisição fora do log, nível de log pra erro esperado, contratos com o Hub e gRPC.
  - `integrations-wizard.md` (novo): carga não é edição, watcher não desfaz ação do usuário, escolha do usuário nunca some em silêncio.
- **`docs/adr/README.md`:** ADR é corrigida no mesmo MR que muda o que ela cita, e regra decidida no review entra na ADR.
- **Template de MR** (`.gitlab/merge_request_templates/Default.md`): formato que o time já usa, com itens fixos em Observações (fora do escopo, removidos e renomeados, fluxos afetados e testados).
- **Apagado de propósito:** árvore de diretórios, mapas de `src/utils/` e `src/constants/`, tabela de scripts de teste, exemplos de código e frases genéricas.
- **Regra alterada:** card obrigatório em todo commit, não só em `feat` e `fix`.

## Onde
partner-portal: `AGENTS.md`, `docs/agents/` (`ui.md`, `testing.md`, `bff.md`, `integrations-wizard.md`), `docs/adr/README.md`, template de MR, `tests/e2e/README.md`. Nenhum arquivo de código mudou.

## Resultado / impacto
O agente lê só o guia do domínio da tarefa, e os pontos repetidos do review viram regra escrita. Todo MR novo do repo já abre com o template.

As regras de `console` no BFF e `data-cy` fixo em `v-for` viraram lint no [PD-93608](../2026-11-16_PD-93608_lint-console-data-cy/README.md) (!544).

## Imagens
Sem prints (só documentação).
