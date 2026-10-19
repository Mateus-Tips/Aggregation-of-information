---
data: 2026-10-19
ticket: MW-91274
ticket_url: https://meagil.atlassian.net/browse/MW-91274
mr:
  - https://gitlab.miisy.me/identity/meweb/me-login-hipster/-/merge_requests/156
projeto: Login Hipster
tipo: observabilidade
camada: [front]
---

# Observabilidade das jornadas de cadastro do Login Hipster

## Resumo
Eventos de negócio nas jornadas de cadastro do login (reset de senha, criação de senha, convite e registro), com tentativa, sucesso e falha com motivo. Continua o padrão das jornadas de acesso (MW-91273).

## Problema
Só as jornadas de acesso (login, SSO, troca de conta, código por email) mandavam eventos. Cadastro, senha e convite ficavam fora do dashboard, sem como ver onde o usuário trava ou por que falha.

## Por que
Fechar o funil por jornada e o painel "Falhas por motivo" do Grafana (observability/dashboards !165), que já leem esses eventos sem mudança no dashboard.

## Como
- **Reset de senha (`PasswordResetForm.vue`):** `password_reset_attempt` depois da validação, `password_reset_success` com rota `email` ou `legacy` (usuário sem migração) e `password_reset_failure` com motivo (`http_404`, `legacy_remote_auth` ou o motivo HTTP).
- **Criação de senha (`pages/password/new.vue`):** `password_create_attempt/success/failure` e `missing_token` quando a página abre sem token.
- **Convite (`pages/invitation/index.vue`):** `invitation_attempt`, `invitation_success` com rota `accepted` ou `registration` e `invitation_failure` com `expired` ou motivo HTTP. A busca do convite ganhou um `.catch` que manda o evento e relança o erro, mantendo o comportamento.
- **Registro (`pages/registration.vue`):** eventos por etapa (`sms_send`, `sms_validate`, `profile`); no `profile` a rota é `password` ou `sso`. URL sem token ou email manda `missing_token` ou `missing_email`.
- **README:** tabela de eventos com as novas jornadas.
- **Testes:** bloco de observabilidade em cada spec, com teste garantindo que senha, código, telefone, nome, email e tokens nunca vão nos atributos.

## Onde
me-login-hipster: `components/PasswordResetForm.vue`, `pages/password/new.vue`, `pages/invitation/index.vue`, `pages/registration.vue`, `README.md`

## Resultado / impacto
Todas as jornadas do login (acesso e cadastro) visíveis no Grafana com funil e motivo de falha, sem dado sensível nos atributos. 28 suítes, 320 testes unitários passando.

## Imagens
Sem prints.
