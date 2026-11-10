---
data: 2026-11-10
ticket: PD-93505
ticket_url: https://me.pro.mercadoe.rocks/pd-93505
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/540
projeto: Partner Portal (iPaaS)
tipo: tech-debt
camada: [front]
---

# Padrão de commit e MR do ME.PRO no partner-portal

## Resumo
Atualizei o `AGENTS.md` do partner-portal pro novo padrão de commit e MR do ME, com a chave do card do ME.PRO no lugar do Jira. Foi o MR de teste pra validar que o hook, o Jenkins e o Atlas leem a chave nova.

## Problema
O `AGENTS.md` ainda mandava usar o padrão do Jira (`Jira MW-XXXXX - #time ...`), que saiu com a migração pro ME.PRO.

## Por que
Os agentes que trabalham no repo precisam gerar commit, título de MR e branch no padrão novo. Também era preciso confirmar que hook, Jenkins e Atlas aceitam a chave nova.

## Como
- **`AGENTS.md`:**
  - commit: `ME.PRO {CARD} - {type}: {description}`;
  - título do MR: `ME.PRO {CARD} - {description}`;
  - branch: `feature/{CARD}`.
- Saem `Jira` e `#time`.
- O próprio MR já seguiu o padrão novo (`ME.PRO PD-93505 - ...`, branch `feature/PD-93505`).

## Onde
partner-portal: `AGENTS.md`

## Resultado / impacto
O partner-portal passa a usar o padrão do ME.PRO. O MR foi mergeado já com a chave nova no commit, no título e na branch.

## Imagens
Sem prints.
