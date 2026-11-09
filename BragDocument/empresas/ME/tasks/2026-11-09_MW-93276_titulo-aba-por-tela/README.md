---
data: 2026-11-09
ticket: MW-93276
ticket_url: https://me.pro.mercadoe.rocks/pd-93276
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/534
projeto: Partner Portal (iPaaS)
tipo: melhoria-ux
camada: [front]
---

# Título da aba do navegador por tela

## Resumo
A aba do navegador mostrava "Partner" em qualquer tela do portal. Agora cada tela tem o próprio título no formato `Tela | Partner Portal`, e as telas de detalhe mostram o nome do registro aberto.

## Problema
Com várias abas do portal abertas, todas tinham o mesmo título, "Partner", e não dava pra saber qual era qual.

## Por que
Achar a aba certa rápido quando há várias abertas.

## Como
- **`nuxt.config`:** `titleTemplate` aplica o sufixo ` | Partner Portal`. Tela sem título próprio mostra só "Partner Portal".
- **Páginas de lista:** `head()` nas 15 telas (integrações, templates, conectores, monitoring, webhooks, API keys, membros, organização, migração e login), com as mesmas chaves i18n do menu.
- **Telas de detalhe:**
  - wizard de integração: `Nome da integração · Etapa`;
  - conector: nome do conector (na criação, o título de novo conector);
  - execução: `Nome do fluxo · Execução`, sem o id.
  No wizard e no conector, o título acompanha o nome enquanto ele é editado, igual ao breadcrumb.
- **i18n:** chaves novas `login.title` e `monitoring.timeline.pageTitle` nos 6 idiomas. `pages.migration` completada em es-ES, es-MX, fr-FR e pt-PT, que antes caíam no inglês, e removida da lista de chaves faltantes do `usedKeys.spec.js`.
- **Testes:** `head()` das 3 telas de detalhe.

## Onde
partner-portal: `nuxt.config`, 15 páginas de lista, wizard de integração, conector, execução do monitoring, arquivos de i18n

## Resultado / impacto
Cada aba mostra a tela e, no detalhe, o nome do registro. `/unauthorized` e as páginas que só redirecionam ficam com "Partner Portal".

## Imagens
Sem prints.
