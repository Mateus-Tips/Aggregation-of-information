---
data: 2026-10-26
ticket: MW-92535
ticket_url: https://meagil.atlassian.net/browse/MW-92535
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/526
projeto: Partner Portal
tipo: bug
camada: [front]
---

# Realinhar campos de destino do outbound quando o payload muda

## Resumo
No wizard do outbound, mudar o payload e voltar pro mapeamento agora atualiza os campos de destino. As ligações dos campos que continuam ficam como estavam. A barra de etapas só navega com a integração ativa, e as trocas de etapa não entram mais no histórico do navegador.

## Problema
- Ao voltar, mudar o payload da estrutura do documento e avançar, o canvas mantinha os campos antigos e descartava o campo novo, mesmo quando a IA sugeria a ligação.
- A barra de etapas deixava pular pra uma etapa concluída sem validar nem salvar a atual. Com isso, dava pra gravar no Hub um mapeamento novo com o template antigo, ou abrir o mapeamento com payload vazio ou inválido.
- O voltar do navegador trocava de etapa sem salvar.

## Por que
O mapeamento precisa refletir o payload salvo, e nenhuma troca de etapa pode pular a validação.

## Como
- **`integrationFlowHydrate.js`:** novo `realignMappingToTargetPayload`, que refaz o `meCatalog` a partir do payload atual e descarta ligações e valores fixos dos campos removidos. Payload igual, vazio ou inválido deixa o mapeamento como está.
- **`IntegrationWizardMappingStep.vue`:** aplica o realinhamento ao entrar no mapeamento do outbound e só emite quando o catálogo mudou, sem disparar nova análise da IA.
- **Barra de etapas:** nova prop `navigable`, ligada só com `flowStatus === 'active'`. Nas outras etapas, a troca é só pelo Voltar e pelo Próximo, que valida e salva. Vale pros 3 fluxos.
- **Histórico (`integrationWizard.js`):** troca de etapa com `$router.replace`, então o voltar do navegador sai do wizard.
- **Decisão de onde realinhar:** o ticket indicava o `afterStepInput`, mas esse hook roda a cada tecla, e o auto-close de chaves gera JSON válido no meio da digitação (no E2E, só 1 de 5 ligações sobreviveu). Por isso o realinhamento roda na entrada do mapeamento, com o payload final.
- **ADR-062:** registra as regras do realinhamento, da barra e do histórico. A ADR-024 ganhou nota dizendo que a regra antiga da barra foi superada.
- **Testes:** helper, entrada no step, barra (com a integração ativa e suspensa) e navegação com `replace`.

## Onde
partner-portal: `integrationFlowHydrate.js`, `IntegrationWizardMappingStep.vue`, `IntegrationWizardSteps.vue`, `IntegrationCreateSubheader.vue`, `layouts/integrations.vue`, `integrationCreateWizardBridge.js`, `integrationWizard.js`, ADR-062, ADR-024

## Resultado / impacto
Mudar o payload do outbound não perde mais o campo novo nem quebra as ligações existentes. O wizard não salva mais estado inconsistente no Hub pela barra ou pelo voltar do navegador. Validado com E2E local contra o Hub do trunk, incluindo os 3 fluxos de criação (`integration-create.spec.js` 5/5).

## Aprendizado
O hook sugerido no ticket parecia o ponto natural, mas o auto-close de chaves gera JSON válido a cada tecla. Testar o cenário real no E2E mostrou que o realinhamento precisava rodar na entrada do mapeamento.

## Imagens
**Item novo no payload (nada removido)**

1. Mapeamento inicial com 5 ligações

![01-mapping-inicial](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/cbabc8157bd964a3408a5afef223402f/01-mapping-inicial.png)

2. Estrutura do documento: entra `moeda`

![02-template-item-novo](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/81fb187e6f0ca6e5072bd6fa26452136/02-template-item-novo.png)

3. Mapeamento: `moeda` sem ligação e as 5 ligações continuam

![03-mapping-item-novo](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/1b2ab19e93b0b431b4de246c5da7a5bc/03-mapping-item-novo.png)

4. Refazer com IA: `moeda` ligado a `order.currency`

![04-mapping-reanalise-2](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/33aadfd694fb7c45e71749f460751e75/04-mapping-reanalise-2.png)

**Campo trocado (sai `dataEntrega`, entra `moeda`)**

5. Estrutura do documento editada

![02-template-editado](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/9ca2b02361dffe209025960eeb08e6f9/02-template-editado.png)

6. Mapeamento: `dataEntrega` sai com a ligação, `moeda` aparece sem ligação e as outras ligações continuam

![03-mapping-apos-troca](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/657995b6946338266c1434accc0c4927/03-mapping-apos-troca.png)
