---
data: 2026-11-04
ticket: MW-92969
ticket_url: https://me.pro.mercadoe.rocks/pd-92969
mr:
  - https://gitlab.miisy.me/identity/me-identity/-/merge_requests/328
projeto: ME Identity
tipo: bug
camada: [back]
---

# Senha, secret, token e telefone mascarados no log dos requests

## Resumo
Quando um command falhava, o Identity gravava o request inteiro no log, inclusive a senha em texto puro. Agora senha, secret, token e telefone aparecem como `***`.

## Problema
O `UnhandledExceptionBehaviour` da `ME.AspNet.Extensions` grava o request no log pelo `ToString()` padrão do record. Por isso, o log de qualquer ambiente podia mostrar a senha em texto puro, e em prod isso já aparecia no Loki.

## Por que
Dado sensível não pode ficar no log. A regra de mascarar só senha, secret, token e telefone foi decisão do tech lead: email, nome, sobrenome, nickname e IDs continuam visíveis.

## Como
- **`SensitiveDataMasker` (Common/Logging):** monta o texto no mesmo formato do record (`Tipo { A = ..., B = ... }`) e troca por `***` todo campo com password, secret, token ou phone no nome. Valor nulo ou vazio aparece como está.
- **Commands que passam a usar o helper no `ToString()`:**
  - usuário: `CreateUserFromInvitation`, `CreateUser`, `CreateUserForOrganization` e `CreateUserPassword` (senha e telefone), `UpdateUser` (telefone);
  - contato da organização: `CreateOrganizationContact` e `UpdateOrganizationContact` (telefone);
  - provider OpenID: `CreateOrganizationProviderOpenId` e `UpdateOrganizationProviderOpenId` (`ClientSecret`).
- **Sem flag nem configuração por ambiente:** vale pra trunk, qa, uat, stg e prod. Os valores reais continuam chegando ao Keycloak; só o texto do log muda.
- **Testes:** `RequestToStringMaskingTests` acha por reflexão os 75 requests da Application, coloca um valor falso nos campos sensíveis e falha se ele aparecer no `ToString()`. Um segundo teste confere o texto exato do `CreateUserFromInvitationCommand`.
- **Fora de propósito:** o `UpdatePasswordExpirationCommand`, cujos campos com "password" no nome são uma flag e um número de dias.

## Onde
me-identity: `Common/Logging/SensitiveDataMasker`, commands de usuário, de contato da organização e de provider OpenID, `RequestToStringMaskingTests`

## Resultado / impacto
O log de erro dos commands do Identity não mostra mais senha, secret, token nem telefone, em nenhum ambiente. Como o teste varre todos os requests, um request novo com campo sensível sem máscara quebra o build.

Fica pendente: o `PhoneService` ainda grava o telefone no texto do log.

## Imagens
Sem prints.
