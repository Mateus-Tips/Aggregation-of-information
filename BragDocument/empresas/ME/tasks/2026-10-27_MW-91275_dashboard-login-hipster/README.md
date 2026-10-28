---
data: 2026-10-27
ticket: MW-91275
ticket_url: https://meagil.atlassian.net/browse/MW-91275
mr:
  - https://gitlab.miisy.me/observability/dashboards/-/merge_requests/165
projeto: Login Hipster
tipo: observabilidade
camada: [infra]
---

# Dashboard do Login Hipster no Grafana

## Resumo
Criei o dashboard do Login Hipster no Grafana. Ele tem as jornadas completas do usuário, a saúde da API e dos pods e os Web Vitals, nas mesmas três seções do dashboard do Partner Portal. Absorveu a MW-91276 (saúde e Web Vitals) e a MW-91302 (backend), que foram canceladas.

## Problema
Os eventos das jornadas de acesso e cadastro (MW-91273 e MW-91274) já chegavam no Loki, mas não havia dashboard pra ver funil, falhas, saúde e performance do login.

## Por que
Dar visibilidade de ponta a ponta do login: onde o usuário trava, por que falha e como estão a API e o front.

## Como
- **Pasta e variáveis:** pasta `Login Hipster` com a tag `login-hipster`. A variável `env` filtra ao mesmo tempo o Faro, o namespace `identity-meweb-<env>` no Mimir e o ambiente dos traces no Tempo.
- **Jornada do Usuário:** sessões, logins e conversão; funil do login (usuário, senha, código); funil do cadastro (convite ou código, SMS, perfil, conta criada); convite, recuperação de senha, SSO e troca de conta; falhas por motivo (top 15) e por jornada.
- **Saúde (Golden Signals):** tráfego, 5xx, 401 e latência p50/p95 por rota do `me-login-api`; CPU e memória dos pods; exceptions do front; logs de erro da API com link pro trace; traces do browser até o `me-login-api`.
- **Performance (Web Vitals):** LCP, INP, CLS e TTFB p75, e LCP/INP por página.
- **Tabela de traces:** a primeira versão filtrava com OU entre `deployment.environment` e `deployment.environment.name`, passava dos 30 s do Grafana ("Bad Request") e trazia chamadas que não são da API. Reescrevi pra buscar os spans de servidor do `me-login-api` cujo trace começa no browser (`trace:rootService`), e a busca passou a levar de 2 a 11 s em até 7 dias.
- **Descrições:** tooltip em português em todos os painéis, explicando o que o painel mostra e como ler.
- **Validação:** conferi as regras do `validate_dashboards.py` com um port em Node, porque não tenho Python local, e rodei todas as queries no Grafana via `/api/ds/query`.

## Onde
observability/dashboards: dashboard JSON do Login Hipster (pasta `Login Hipster`)

## Resultado / impacto
O login tem um dashboard completo, igual ao do Partner Portal, com funil por jornada, motivo de falha, saúde da API e dos pods e Web Vitals. Isso fecha a frente de observabilidade do login (MW-91273, MW-91274, MW-91954 e este dashboard).

## Aprendizado
O filtro com OU no TraceQL estourava o timeout em ambiente com pouco tráfego. Partir do span de servidor com `trace:rootService` fez a busca cair de mais de 30 s pra 2 a 11 s e trazer só as chamadas da API.

## Imagens
Jornada do Usuário

![ev-1-jornada](https://gitlab.miisy.me/observability/dashboards/uploads/9f98125036512d6a6cfcedab0faeb981/ev-1-jornada.png)

Saúde (Golden Signals)

![ev-2-saude](https://gitlab.miisy.me/observability/dashboards/uploads/7ecd5415a935aa80cc1bcb64ea71c52c/ev-2-saude.png)

Performance (Web Vitals)

![ev-3-performance](https://gitlab.miisy.me/observability/dashboards/uploads/1f917a68bf9fcb6e8220c09e57952e87/ev-3-performance.png)
