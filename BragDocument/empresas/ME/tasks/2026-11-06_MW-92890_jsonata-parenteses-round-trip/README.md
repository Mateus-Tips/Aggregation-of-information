---
data: 2026-11-06
ticket: MW-92890
ticket_url: https://me.pro.mercadoe.rocks/pd-92890
mr:
  - https://gitlab.miisy.me/partner-ecosystem/integration-hub/-/merge_requests/298
projeto: Partner Portal (iPaaS)
tipo: bug
camada: [back]
---

# Serializador JSONata para de acrescentar parênteses a cada salvar

## Resumo
A expressão JSONata com bloco entre parênteses ganhava um par de parênteses a cada salvar e a cada abertura da integração. Agora o texto fica estável, e o que já foi gravado com camadas extras volta à forma simples no próximo salvar.

## Problema
O Hub reescreve o texto da expressão a cada salvar e a cada abertura. Num passo de path em bloco, como `Itens.(Qtd * Preco)`, o serializador acrescentava parênteses toda vez. O valor calculado não mudava, mas o texto crescia, e o portal fazia PATCH ao abrir a integração mesmo sem edição.

## Por que
Era o bug deixado de fora na [MW-92706](../2026-11-04_MW-92706_compose-objeto-unico-projecao/README.md): o texto da expressão não pode mudar sozinho a cada ida e volta.

## Como
- **`JsonataSourceWriter.WriteStep`:** saiu o `wrapNonName`, que duplicava o parêntese que o `WriteBlock` já coloca. Função como passo (`Itens.$string(Qtd)`) também não ganha mais parênteses. O `WriteName` saiu junto.
- **`JsonataSourceWriter.WriteBlock`:** bloco cujo único item é outro bloco sem filtro é escrito uma vez só, então `Itens.(( ( Qtd * Preco ) ))` volta pra `Itens.( Qtd * Preco )`. Bloco com mais de uma expressão ou com filtro fica como está, e o escopo de variável não muda.
- **Passaram a serializar:** passo array (`Itens.[Qtd, Preco]`), que antes era recusado porque o parêntese extra mudava o resultado, e menos unário (`-Qtd`, `-(Qtd + 1)`, `2 * -Qtd`), que falhava com `UnaryMinusNode` não suportado.
- **`MappingAstService.CollectPaths`:** o `Sources` do decompose agora olha dentro do menos unário e do passo que não é nome. Antes essas expressões vinham sem origem, e o canvas ficava sem a linha.
- **Testes:** parse e serialize 3 vezes devolvem o mesmo texto; a avaliação é igual antes e depois (com `:=`, filtro, passo array e menos unário); `Sources` certo no decompose; compose, decompose e compose de novo devolvem o mesmo mapping; mapping com camadas extras sai na forma simples.

## Onde
integration-hub: `JsonataSourceWriter`, `MappingAstService`, testes do serializador e do compose

## Resultado / impacto
- O texto das expressões não cresce mais, e abrir uma integração não gera PATCH sem edição. A integração já gravada com camadas extras dispara um PATCH só na primeira abertura, pela normalização.
- O canvas passa a mostrar a linha de origem pra expressões com bloco, função, passo array e menos unário.
- `Itens.(( X ))` e `Itens.( X )` agora contam como iguais no compose e não geram mais `leaf_conflict`.
- Sem mudança de contrato nem de front.

## Imagens
Sem prints.
