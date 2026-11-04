---
data: 2026-11-04
ticket: MW-92706
ticket_url: https://me.pro.mercadoe.rocks/pd-92706
mr:
  - https://gitlab.miisy.me/partner-ecosystem/integration-hub/-/merge_requests/297
projeto: Partner Portal (iPaaS)
tipo: bug
camada: [back]
---

# Compose junta objeto único com a projeção da mesma lista de destino

## Resumo
Quando dois campos da mesma lista de destino chegavam em formas diferentes, o compose descartava um deles sem avisar. Agora ele junta os dois na mesma projeção e não perde campo.

## Problema
Um campo vinha como projeção sobre uma lista da origem (`[PedidoCompra.Itens.{ ... }]`) e outro como objeto único (`[{ 'ref': PedidoCompra.C7_NUM }]`). O compose ficava com um, descartava o outro com `structural_conflict` e não mostrava erro na tela.

## Por que
O usuário perdia campo do mapeamento sem saber, tanto no mapeamento automático quanto na transformação.

## Como
- **`IMappingAstService.AnchorPathsAtRoot`:** devolve uma cópia do leaf com cada path absoluto prefixado por `$$`, inclusive dentro de função, concat, condicional e bloco. `$$` e `$var` ficam como estão. Quando a reescrita não é segura (iterator próprio, lambda, `$` ou `$.X`), devolve `null`.
- **`MappingComposer.Merge`:** o objeto único (`[{...}]` ou `{...}`) ao lado de uma projeção (`[Src.{...}]`, `Src[].{...}` ou `Src.{...}`) vira iterator com a mesma origem e entra no merge que já existia, nas duas ordens. Colisão de chave segue o `leaf_conflict`. Em lista aninhada, só junta se o campo não tiver path relativo.
- **Conflitos mantidos:** reescrita não segura continua `structural_conflict`, e duas projeções sobre listas diferentes continuam `iterator_path_mismatch`.
- **Testes:** `MappingComposerSingleObjectIntoProjectionTests` cobre as duas ordens, formas de envelope, função e concat, colisão, casos não seguros, lista aninhada e round-trip compose/decompose com `sources` preservado. Também há casos novos em `MappingAstServiceTests`.

O exemplo vira `{ 'items': [PedidoCompra.Itens.{ 'code': C7_X, 'ref': $$.PedidoCompra.C7_NUM }] }`.

## Onde
integration-hub: `MappingAstService`, `MappingComposer`, testes do compose e da AST

## Resultado / impacto
O mapeamento não perde mais campo nesse caso e não gera warning. Vale pro mapeamento automático e pra transformação, sem mudança de contrato nem de front. De quebra, um `[{}]` vazio ao lado de uma projeção não apaga mais a projeção, o que antes acontecia dependendo da ordem.

Fica na MW-92890: o `JsonataSourceWriter` acrescenta parênteses em `Itens.(...)` a cada compose/decompose.

## Imagens
Sem prints.
