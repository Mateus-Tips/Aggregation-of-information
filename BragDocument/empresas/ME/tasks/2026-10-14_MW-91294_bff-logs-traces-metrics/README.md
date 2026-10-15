---
data: 2026-10-14
ticket: MW-91294
ticket_url: https://meagil.atlassian.net/browse/MW-91294
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/517
projeto: Partner Portal (iPaaS)
tipo: observabilidade
camada: [bff]
---

# Log do BFF ligado ao trace e endpoint /metrics

## Resumo
O log do BFF do Partner Portal agora leva ao trace no Tempo, e o BFF expõe `/metrics` com latência por rota e métricas do Node, entrando no HTTP Overview e no SLO.

## Problema
O BFF já entrava no trace, mas o log no Loki não levava ao trace no Tempo. O alvo `/metrics` do pod (`job="partner-portal"`) estava down, então o BFF ficava fora do HTTP Overview, do SLO e sem métricas de saturação do Node.

## Por que
Ver o BFF nos mesmos painéis e SLOs dos outros serviços e ir do log direto pro trace na investigação.

## Como
- **Logger (`src/utils/shared/logger.js`):** o log JSON grava `trace_id` e `span_id` do span ativo. O Alloy promove o `trace_id` pra metadado do Loki e o Grafana abre o trace a partir da linha de log.
- **Métricas (`src/api/metrics.js`):** serverMiddleware com `prom-client`, registrado primeiro:
  - Histograma `http_server_request_duration_seconds` nas rotas `/api/*`, com método, rota e status. Mesmo contrato das rules `me_http_*` e mesmos buckets dos serviços .NET.
  - `http_route` é o mount do handler + rota do Express (ex.: `/api/integrations/flows/:id`), sem IDs, pra não explodir cardinalidade. Requisição sem handler vira `unmatched`.
  - Métricas padrão do Node (CPU, heap, event loop, GC).
  - `/metrics` só responde ao coletor interno; requisição com `X-Forwarded-For` (ingress público) recebe 404.
- A porta 8080 já é coletada pela plataforma, sem mudança no Helm.
- Testes do logger e do middleware (rótulo de rota, rotas fora de `/api`, 404 externo).

## Onde
partner-portal: `src/utils/shared/logger.js`, `src/api/metrics.js`, `nuxt.config.js`, `package.json`

## Resultado / impacto
BFF no ME Ops / HTTP Overview e Application Overview, com SLO e saturação do Node. Do log no Loki dá pra abrir o trace no Tempo. Critérios de aceite dependem do deploy no trunk (`up=1` no `job="partner-portal"`).

## Aprendizado
Seguir o mesmo contrato de métricas dos outros serviços (nome, rótulos, buckets) faz o serviço entrar nos dashboards e SLOs prontos sem trabalho extra.

## Imagens
Sem prints.
