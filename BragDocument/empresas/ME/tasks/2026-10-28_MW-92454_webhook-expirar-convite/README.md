---
data: 2026-10-28
ticket: MW-92454
ticket_url: https://meagil.atlassian.net/browse/MW-92454
mr:
  - https://gitlab.miisy.me/identity/me-identity/-/merge_requests/326
projeto: ME Identity
tipo: bug
camada: [back]
---

# Webhook de expirar convite ignora convite que não está mais pendente

## Resumo
O webhook que expira convites no Identity agora ignora convites que já foram aceitos, cancelados ou expirados. Ele responde 202 em vez de 422, e isso tirou 97% dos logs de Error do Identity em prod.

## Problema
O Dispatcher chama o webhook de expirar convite 7 dias depois da criação. Quando o convite já tinha sido aceito, cancelado ou expirado, a API devolvia 422, e o Dispatcher tentava de novo até 10 vezes. Cada tentativa gerava um log de Error. Em 7 dias de prod, 97% dos logs de Error do Identity vinham desse webhook.

## Por que
Tirar o ruído dos logs de Error do Identity, pra que erro de verdade apareça, e parar os retries inúteis do Dispatcher.

## Como
- **`ExpireInvitationWebhookHandler`:** quando o convite não está pendente, retorna sem chamar `Expire()`, que lançava `RuleViolationDomainException`. Nesse caso loga em Information o `InvitationId` e o `Status`, no mesmo padrão do `ExpireUserPasswordHandler`.
- **Testes do handler:** os casos de convite aceito, cancelado e expirado agora esperam que não haja exceção nem save.
- **Testes que não validavam nada:** os testes antigos não tinham `await` no `ThrowAsync`, então passavam sem validar a asserção. Corrigi, incluindo o caso de convite não encontrado.

## Onde
me-identity: `ExpireInvitationWebhookHandler`, testes do handler

## Resultado / impacto
Cerca de 97% dos logs de Error do Identity em prod deixam de existir, e o Dispatcher não refaz mais as 10 tentativas por convite. Quem chama `IInvitationClient.ExpireAsync` na Library pra um convite que não está pendente passa a receber 202 em vez de 422.

## Aprendizado
`ThrowAsync` sem `await` passa sempre. Vale procurar esse padrão nos outros testes do repo.

## Imagens
Sem prints.
