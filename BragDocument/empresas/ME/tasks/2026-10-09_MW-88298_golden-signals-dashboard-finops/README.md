---
data: 2026-10-09
ticket: MW-88298
ticket_url: https://meagil.atlassian.net/browse/MW-88298
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/510
  - https://gitlab.miisy.me/observability/dashboards/-/merge_requests/161
projeto: Partner Portal (iPaaS) + observability/dashboards
tipo: observabilidade
camada: [front, bff]
---

# Golden Signals do Wizard v2: spans por rota, métricas do Hub e dashboard FinOps

## Resumo
Front e BFF agora geram os dados de Golden Signals do Wizard v2 (spans das rotas de IA com a rota no nome e medição de toda requisição ao Hub), e o dashboard `Partner Portal / Wizard FinOps` foi reorganizado em Jornada do Usuário, Saúde e Performance.

## Problema
Os spans do BFF saíam só como `POST`/`GET`, sem a rota, então não dava pra separar latência e erro por endpoint de IA. O front não media as chamadas ao Hub, e as rotas de IA logavam com `console.error`.

## Por que
Base pros Golden Signals (latência, tráfego, erro) do dashboard FinOps do Wizard v2.

## Como
- **BFF, middleware `nameServerSpanByRoute`:** renomeia o span do OTel pra `MÉTODO /rota` só quando o caminho está na lista de rotas conhecidas da API. Caminho fora da lista mantém o nome padrão, então a cardinalidade fica limitada. Registrado antes do `bodyParser` e do `extractToken`, então 400, 401 e 413 também caem no span certo.
- **BFF, rotas de mapping e transformation:** passam as rotas conhecidas e trocam `console.error` por `createLogger`. O log de mappings inclui o `hubPath` que falhou.
- **Front, plugin `hub-request-observability`:** mede duração e resultado de toda requisição a `/api/integrations/*` e envia `partner_integration_hub_request` ao Faro com método, rota normalizada, resultado e classe do status (`2xx`, `4xx`, `5xx`, `timeout`, `network`). Em retry após refresh de token, a medição parte da primeira tentativa.
- **Front, `hubRequestRoute`:** normaliza a rota por lista de segmentos conhecidos; segmento novo vira `:id`.
- **Dashboard (observability/dashboards !161):** `Partner Portal / Wizard FinOps` reorganizado em três seções. **Jornada do Usuário:** funil por etapa, conversão, tempo até a ativação, chamadas de IA e integrações criadas por ERP e tipo de documento. **Saúde:** tráfego, erros e latência do BFF e das chamadas ao Hub, CPU e memória dos pods, exceptions. **Performance:** Core Web Vitals das telas do wizard. O JSON no repo é o export da versão publicada no Grafana (uid `partner-portal-finops`).
- Testes do middleware (incluindo chamadas HTTP reais provando o nome do span em 401 e 400), do plugin e da normalização.

## Onde
partner-portal: `src/api/utils.js`, `src/api/integrations-mappings.js`, `src/api/integrations-transformations.js`, `src/plugins/hub-request-observability.js`, `src/utils/integrations/hubRequestRoute.js`

observability/dashboards: `files/Partner Portal/frontend-wizard.json`

## Resultado / impacto
O dashboard mostra a jornada do usuário no wizard, a saúde do BFF e do Hub (latência e erro por rota de IA, com cardinalidade controlada) e os Web Vitals. Os painéis do BFF por rota dependem do !510 chegar ao ambiente; antes disso ficam em "No data".

Fora de escopo: o Faro descarta medições idênticas em sequência (dedupe padrão), o que também afeta `partner_integration_ai_calls`.

## Aprendizado
Em métricas e spans, sempre limitar cardinalidade: nomear só rotas conhecidas e normalizar ids pra `:id`.

## Imagens
Prints do MR !161 (stg, últimos 30 dias), abrem logado no GitLab:
- [Jornada do Usuário](https://gitlab.miisy.me/observability/dashboards/uploads/8ad8440c38f4d267fb523191a0636882/finops-1.png)
- [Saúde: tráfego, CPU, memória e Hub](https://gitlab.miisy.me/observability/dashboards/uploads/b8f5205d8be5f23873829963b964b2c1/finops-2.png)
- [Saúde: exceptions, memória do browser](https://gitlab.miisy.me/observability/dashboards/uploads/53defe7a10cc53ccd20ec3f469a9d9d1/finops-3.png)
- [Chamadas por destino e Web Vitals](https://gitlab.miisy.me/observability/dashboards/uploads/2aa4cba3c25f9d1b30cb46d226bec299/finops-4.png)
