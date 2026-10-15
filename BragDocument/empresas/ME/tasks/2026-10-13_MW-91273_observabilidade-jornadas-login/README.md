---
data: 2026-10-13
ticket: MW-91273
ticket_url: https://meagil.atlassian.net/browse/MW-91273
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/147
  - https://gitlab.miisy.me/devops/helm/identity/meweb/-/merge_requests/68
projeto: Login Hipster
tipo: observabilidade
camada: [front, infra]
---

# Observabilidade das jornadas de acesso do Login Hipster

## Resumo
O Login Hipster passou a mandar eventos de negócio das jornadas de acesso (login, SSO, troca de conta, código por email) pro Grafana, expõe `/metrics` pro Prometheus e envia o nome certo do ambiente (antes tudo chegava como `production`).

## Problema
Não havia eventos das jornadas de acesso pra montar dashboard. O Prometheus marcava o app como fora do ar (`up=0`) porque raspava uma rota `/metrics` que não existia, os spans do servidor saíam como `nuxt-app` e todos os ambientes chegavam como `production`.

## Por que
Alimentar o dashboard do Grafana com tentativa, sucesso e falha (com motivo) de cada jornada, e conseguir separar os dados por ambiente.

## Como
**App (me-login-hipster !147):**
- **`Dockerfile`:** `NUXT_PUBLIC_OBSERVABILITY_SERVICE_NAME=me-login-hipster`, igual ao partner-portal. Sem isso os spans do servidor saem como `nuxt-app`.
- **`/metrics` (`src/api/metrics.js`):** métricas padrão do Node com `prom-client`, servidas na raiz, fora do `/login` que o ingress expõe.
- **Helper (`src/utils/observability.js`):** `errorReason` gera o motivo (`http_<status>`, `network_error`, `client_error`) e `withSpan` roda a chamada uma vez só, com ou sem tracing.
- **Eventos e spans:** login (`login_attempt/success/failure` com rota `meid`, `legacy`, `password`, `email`, `blocked`), SSO, troca de conta (`setUser` só com id interno) e código por email.
- **Troca de conta:** `return` depois dos redirects do `asyncData`, evitando um GET com `guid=undefined` e um segundo redirect.
- **Testes:** eventos de cada fluxo e garantia de que username, email, senha, token e código nunca vão nos atributos.

**Helm (devops/helm/identity/meweb !68):**
- **`values.yaml` (`login-hipster.extraEnvVars`):** `NUXT_PUBLIC_OBSERVABILITY_ENVIRONMENT` com o `global.env`. Vale pra todos os ambientes, porque os `values.<env>.yaml` só sobrescrevem o `API_BASE_URL`.
- Mesmo padrão do partner-portal: no Helm só o que muda por ambiente. O nome do serviço fica no `Dockerfile` do app e a observabilidade já vem ligada por padrão.

## Onde
me-login-hipster: `Dockerfile`, `nuxt.config.js`, `src/api/metrics.js`, `src/utils/observability.js`, `pages/index.vue`, `pages/sso/_alias.vue`, `pages/switch.vue`, `pages/email.vue`

devops/helm/identity/meweb: `values.yaml`

## Resultado / impacto
Jornadas de acesso visíveis no Grafana por ambiente, com motivo de falha e sem dado sensível nos atributos. Prometheus passa a ver o app no ar (`up=1`) depois do deploy.

## Imagens
Sem prints.
