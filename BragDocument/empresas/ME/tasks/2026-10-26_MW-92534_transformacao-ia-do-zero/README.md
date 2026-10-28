---
data: 2026-10-26
ticket: MW-92534
ticket_url: https://meagil.atlassian.net/browse/MW-92534
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/527
projeto: Partner Portal (iPaaS)
tipo: melhoria-ux
camada: [front]
---

# Geração de transformação com IA do zero, com campo de origem e hover no badge

## Resumo
No modal de transformação do mapeamento, cada geração com IA começa do zero e leva o campo de origem da conexão. O hover no badge destaca a conexão, e o modal abre com foco no prompt, que virou o campo principal da tela.

## Problema
- O front mandava a `currentExpression`, então uma geração errada ou com falha prendia o usuário na primeira expressão.
- A IA não sabia qual campo de origem estava ligado.
- O hover no badge de transformação não destacava a conexão como o hover da linha. No modal, prompt, expressão e prévia tinham o mesmo peso visual.

## Por que
Deixar a geração de transformação previsível e o modal mais fácil de usar.

## Como
- **`MappingTransformModal` (request):** remove `currentExpression` e envia `sourceField` sem o prefixo `$.`. Com várias origens (concat), vai só o primeiro campo, e os outros o usuário cita na descrição.
- **`MappingTransformModal` (foco e hierarquia):** foco no prompt ao abrir, fora do modo leitura. Prompt e expressão são os únicos campos editáveis: o prompt fica em destaque e a expressão fica compacta, em monospace e sem roxo. A prévia de entrada e saída virou um painel somente leitura, e os rótulos saíram do uppercase.
- **`MappingCanvas`:** o badge aplica `setHighlight` no `mouseenter` (mesmo caminho do hover da linha) e limpa no `mouseleave`. Ganhou `data-cy="mapping-transform-badge"`.
- **Testes:** request sem `currentExpression` e com `sourceField` (origem simples, várias origens e sem origem), foco no prompt e hover do badge.
- **Contrato:** depende do `sourceField` do integration-hub (MW-92533). A ordem de deploy é livre, porque o .NET ignora propriedade desconhecida.

## Onde
partner-portal: `MappingTransformModal.vue`, `MappingCanvas.vue`

## Resultado / impacto
Gerar de novo sempre parte do zero, sem herdar uma expressão errada. A IA recebe o campo de origem, e o modal deixa claro onde o usuário escreve.

## Imagens
Modal abre com o cursor no prompt; expressão compacta, sem roxo e sem uppercase

![07-modal-open](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/6c78baccc13c9e35e7bd030d4afc89b9/07-modal-open.png)

Geração com `sourceField` e sem `currentExpression`; prévia como painel somente leitura

![08-modal-generated](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/8bf87d1f23bfbb4e3235affc01343682/08-modal-generated.png)
