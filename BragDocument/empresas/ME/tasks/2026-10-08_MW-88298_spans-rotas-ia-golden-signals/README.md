---
data: 2026-10-08
ticket: MW-88298
ticket_url: https://meagil.atlassian.net/browse/MW-88298
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/510
projeto: Partner Portal (iPaaS)
tipo: observabilidade
camada: [front, bff]
---

# Spans nomeados por rota e métricas do Hub para os Golden Signals

## Resumo
Front e BFF agora geram os dados de Golden Signals do dashboard FinOps do Wizard v2: spans das rotas de IA do BFF com a rota no nome e medição de toda requisição do front ao Hub.

## Problema
Os spans do BFF saíam só como `POST`/`GET`, sem a rota, então não dava pra separar latência e erro por endpoint de IA. O front não media as chamadas ao Hub, e as rotas de IA logavam com `console.error`.

## Por que
Base pros Golden Signals (latência, tráfego, erro) do dashboard FinOps do Wizard v2.

## Como
- **BFF, middleware `nameServerSpanByRoute`:** renomeia o span do OTel pra `MÉTODO /rota` só quando o caminho está na lista de rotas conhecidas da API. Caminho fora da lista mantém o nome padrão, então a cardinalidade fica limitada. Registrado antes do `bodyParser` e do `extractToken`, então 400, 401 e 413 também caem no span certo.
- **BFF, rotas de mapping e transformation:** passam as rotas conhecidas e trocam `console.error` por `createLogger`. O log de mappings inclui o `hubPath` que falhou.
- **Front, plugin `hub-request-observability`:** mede duração e resultado de toda requisição a `/api/integrations/*` e envia `partner_integration_hub_request` ao Faro com método, rota normalizada, resultado e classe do status (`2xx`, `4xx`, `5xx`, `timeout`, `network`). Em retry após refresh de token, a medição parte da primeira tentativa.
- **Front, `hubRequestRoute`:** normaliza a rota por lista de segmentos conhecidos; segmento novo vira `:id`.
- Testes do middleware (incluindo chamadas HTTP reais provando o nome do span em 401 e 400), do plugin e da normalização.

## Onde
partner-portal: `src/api/utils.js`, `src/api/integrations-mappings.js`, `src/api/integrations-transformations.js`, `src/plugins/hub-request-observability.js`, `src/utils/integrations/hubRequestRoute.js`

## Resultado / impacto
O dashboard consegue mostrar latência e erro por rota de IA no BFF e por chamada ao Hub no front, com cardinalidade controlada.

Fora de escopo: o Faro descarta medições idênticas em sequência (dedupe padrão), o que também afeta `partner_integration_ai_calls`.

## Aprendizado
Em métricas e spans, sempre limitar cardinalidade: nomear só rotas conhecidas e normalizar ids pra `:id`.

## Imagens
Sem prints.
