---
data: 2026-11-06
ticket: MW-93072
ticket_url: https://me.pro.mercadoe.rocks/pd-93072
mr:
  - https://gitlab.miisy.me/identity/me-identity/-/merge_requests/329
projeto: ME Identity
tipo: bug
camada: [back]
---

# Telefone fora do log de erro da validação de telefone

## Resumo
Quando a validação de telefone falhava, o `PhoneService` gravava o número completo no log. Agora o log mostra só o IDD e a exceção.

## Problema
O `LogError` no `catch` de `NumberParseException` do `PhoneService` recebia o número de telefone. Era o último ponto que ficou de fora da [MW-92969](../2026-11-04_MW-92969_mascarar-dados-sensiveis-log/README.md).

## Por que
Fechar o critério da PD-92968: nenhum log do Identity pode mostrar telefone em texto puro.

## Como
- **`PhoneService`:** o `LogError` não recebe mais o número. Ficam o IDD (`{Idd}`) e a exceção, cujas mensagens são textos fixos da libphonenumber, sem o número.
- **`PhoneServiceTests`:** teste novo que falha se o número aparecer no log quando o parse falha.
- Sem flag: vale pra trunk, qa, uat, stg e prod.

## Onde
me-identity: `PhoneService`, `PhoneServiceTests`

## Resultado / impacto
Junto com a MW-92969, nenhum log do Identity mostra mais telefone em texto puro.

## Imagens
Sem prints.
