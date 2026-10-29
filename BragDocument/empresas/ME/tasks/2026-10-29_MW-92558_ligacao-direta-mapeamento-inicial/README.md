---
data: 2026-10-29
ticket: MW-92558
ticket_url: https://meagil.atlassian.net/browse/MW-92558
mr:
  - https://gitlab.miisy.me/partner-ecosystem/integration-hub/-/merge_requests/292
projeto: Partner Portal (iPaaS)
tipo: melhoria-ux
camada: [back]
---

# Ligação direta na geração inicial do mapeamento quando o campo só formata

## Resumo
Na geração inicial do mapeamento com IA, o campo cuja expressão lê um único campo de origem e só formata texto ou faz conta agora vira ligação direta. A garantia fica no código, sem mudar o prompt.

## Problema
Na primeira análise, a IA devolvia expressões como `$uppercase(Campo)` ou `$trim(Campo)` em campos que o usuário veria melhor como uma ligação simples entre origem e destino. No canvas, isso aparecia como transformação em vez de ligação direta.

## Por que
O mapeamento inicial fica mais simples de ler e de editar: o que é só formatação de um campo aparece como ligação, e o que muda o valor de verdade continua como expressão.

## Como
- **`DirectLinkConverter`:** desce pelo envelope (objeto de uma entrada, projeção, array de um elemento), igual ao `MappingKindClassifier`, e troca o leaf pelo path quando ele lê um único campo de origem. O prefixo `$$.` é preservado.
- **Allowlist:** só convertem `$uppercase`, `$lowercase`, `$trim`, `$replace`, `$substring`, `$substringBefore`, `$substringAfter`, `$pad`, `&` e conta. Na condição do ternário, qualquer função é aceita (ex.: guarda com `$exists`).
- **Fica como a IA gerou:** coerção de tipo, agregação, comparação, de/para, fallback fixo, sufixo com `&`, expressão com 2 ou mais campos de origem, literal e path. AST inesperada mantém o campo original.
- **`MappingAgent`:** aplica a conversão depois do parse e de novo depois do reparo, só no modo Initial. Com instruction (chat) ou com mapeamento atual, nada muda. Vale também pro connector request.
- **Testes:** casos que viram path e casos que ficam como a IA gerou, incluindo kind e sources recalculados no handler. Com `$$`, o kind continua `expression` pra não mudar o round-trip de integrações existentes.

## Onde
integration-hub: `DirectLinkConverter`, `MappingAgent`, testes do agente e do `GenerateMappingHandler`

## Resultado / impacto
O mapeamento inicial chega com mais ligações diretas e menos transformações desnecessárias, sem depender de a IA seguir o prompt. O chat e a reanálise com mapeamento atual não mudam.

## Imagens
Sem prints.
