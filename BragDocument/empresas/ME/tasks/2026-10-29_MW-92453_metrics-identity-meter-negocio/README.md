---
data: 2026-10-29
ticket: MW-92453
ticket_url: https://meagil.atlassian.net/browse/MW-92453
mr:
  - https://gitlab.miisy.me/identity/me-identity/-/merge_requests/327
projeto: ME Identity
tipo: observabilidade
camada: [back]
---

# `/metrics` duplicado no Identity e métricas de negócio exportadas

## Resumo
Removi o endpoint `/metrics` duplicado do Identity e passei a exportar o meter de negócio `me_id_metrics` (usuário criado, atualizado, excluído etc.), que nunca tinha chegado ao Prometheus.

## Problema
- O `Startup.cs` registrava um `/metrics` próprio do OpenTelemetry antes do routing, além do que a `ME.AspNet.Extensions` já registra.
- O meter `me_id_metrics` não estava em `OpenTelemetryConfiguration.MeterNames`, então as métricas de negócio nunca foram exportadas.

## Por que
Ter as métricas de negócio do Identity no Grafana e deixar o `/metrics` com uma única origem.

## Como
- **`WebApi/Startup.cs`:** remove o `UseOpenTelemetryPrometheusScrapingEndpoint()`. O `/metrics` continua respondendo pelo endpoint da lib.
- **`WebApi/appsettings.json`:** `"MeterNames": ["me_id_metrics"]` em `OpenTelemetryConfiguration`.
- **Testes de integração:** novo `BusinessMetricsTests`, que registra um `UserCreated()` e confere que `me_id_user_created` aparece no `/metrics`. Sem o `MeterNames`, o teste falha.

## Onde
me-identity: `WebApi/Startup.cs`, `WebApi/appsettings.json`, `BusinessMetricsTests`, `BaseIntegrationTest`

## Resultado / impacto
As métricas de negócio do Identity passam a chegar no Prometheus (ex.: `me_id_user_created_total`), e o `/metrics` tem uma origem só. O scrape aparecia como cerca de 40% do tráfego HTTP do serviço; vale conferir essa participação no Grafana depois do deploy em stg e prod.

## Imagens
Sem prints.
