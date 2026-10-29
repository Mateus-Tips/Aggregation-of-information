---
data: 2026-10-28
ticket: MW-92533
ticket_url: https://meagil.atlassian.net/browse/MW-92533
mr:
  - https://gitlab.miisy.me/partner-ecosystem/integration-hub/-/merge_requests/289
projeto: Partner Portal (iPaaS)
tipo: feature
camada: [back]
---

# Agente de transformação com campo de origem e sem variáveis

## Resumo
O agente de transformação do Integration Hub (`POST /v1/transformations`) passou a receber o `sourceField` e a gerar cada expressão do zero. Ele também deixou de aceitar variáveis, atribuições e blocos na expressão gerada. É o back da MW-92534, que fez a mesma mudança no front.

## Problema
- O agente recebia a `currentExpression`, então uma geração errada prendia o usuário na primeira expressão.
- A IA não sabia qual campo de origem estava ligado ao alvo.
- A IA podia gerar expressões com variável e bloco, fora do subset que o `JsonataSourceWriter` sabe escrever. O cookbook anexado ao prompt usava variáveis, contradizendo a regra.

## Por que
Deixar a geração previsível e garantir que a expressão gerada caiba no subset que o mapeamento consegue reescrever.

## Como
- **Contrato (`GenerateTransformationCommand`):** entra o `sourceField` opcional e sai a `currentExpression`. Um request antigo continua funcionando, e o campo antigo é ignorado.
- **Validator:** o `sourceField` precisa existir no `source` (400 caso contrário). O validator entende `[]` de lista: `Itens[].C7_X` procura `C7_X` nos itens.
- **AST (`FindVariableOrBlock`):** acha bloco, atribuição, variável e definição de função, inclusive dentro de filtro e lambda, e recusa nó fora do subset (`~>`, ordenação, transform). Continuam aceitos o parêntese de agrupamento, o `$$` e as funções nativas.
- **Repair loop (`ExpressionRepairer`):** ganhou uma checagem extra opcional depois do dry-run. A LLM tenta corrigir a expressão, e se não conseguir, o agente devolve 422. Mapeamento e preview não mudam.
- **Prompt (`transformation-core.md`):** usa o `sourceField` como base, lista variável, atribuição e bloco como proibidos e ganhou few-shots novos. O snapshot foi atualizado.
- **Cookbook:** exemplos de data e de dicionário reescritos sem variável, com a mesma expressão resultante.
- **Docs:** `swagger.json` e doc do controller atualizados.
- **Testes:** Application 1295/1295, Infrastructure 556/556, Domain 690/690 e WebApi.Unit 77/77.
- **Teste com LLM real:** rodei o handler com a DI real do Hub (OpenRouter, Claude Haiku). Testei campo escalar, item de lista com `[]`, concat citado na descrição, `sourceField` inexistente (400), pedido explícito de variável (gerou sem `:=`) e `$$` da raiz dentro do item.

## Onde
integration-hub: `GenerateTransformationCommand`, validator, `IMappingAstService`, `ExpressionRepairer`, `MappingAgent`, `transformation-core.md`, `jsonata-cookbook.md`, `TransformationsController`, `swagger.json`

## Resultado / impacto
Gerar de novo sempre parte do zero, a IA sabe qual campo foi ligado, e a expressão gerada sempre cabe no que o mapeamento consegue escrever. A ordem de deploy com o front (partner-portal !527) é livre, porque o .NET ignora propriedade desconhecida.

## Aprendizado
A lib JSONata transforma todo parêntese em `BlockNode`, então proibir "bloco" de forma ingênua barrava `(A + B) * 2`. A checagem precisou diferenciar o bloco com várias expressões do parêntese de agrupamento.

## Imagens
Sem prints.
