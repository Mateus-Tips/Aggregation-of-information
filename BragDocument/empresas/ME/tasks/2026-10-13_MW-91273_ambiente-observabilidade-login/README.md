---
data: 2026-10-13
ticket: MW-91273
ticket_url: https://meagil.atlassian.net/browse/MW-91273
mr:
  - https://gitlab.miisy.me/devops/helm/identity/meweb/-/merge_requests/68
projeto: Login Hipster
tipo: observabilidade
camada: [infra]
---

# Ambiente certo na observabilidade do Login Hipster

## Resumo
O Login Hipster passou a mandar o nome do ambiente (trunk, qa, stg, uat, prod) pro Grafana; antes tudo chegava como `production`.

## Problema
A variável de ambiente da observabilidade não existia no Helm, então eventos e traces de todos os ambientes chegavam como `production`.

## Por que
Sem separar por ambiente, não dá pra filtrar dados de stg ou trunk no Grafana nem confiar nos números de prod.

## Como
- **`values.yaml` (`login-hipster.extraEnvVars`):** `NUXT_PUBLIC_OBSERVABILITY_ENVIRONMENT` com o `global.env`. Vale pra todos os ambientes, porque os `values.<env>.yaml` só sobrescrevem o `API_BASE_URL`.
- Mesmo padrão do partner-portal: no Helm só o que muda por ambiente. O nome do serviço fica no `Dockerfile` do app e a observabilidade já vem ligada por padrão.

## Onde
devops/helm/identity/meweb: `values.yaml`

## Resultado / impacto
Eventos e traces do Login Hipster aparecem no Grafana com o ambiente certo.

## Imagens
Sem prints.
