# Roteiro da review 2026-11-03 (~13 min + 2 de folga)

Período 2026-10-07 → 2026-11-03. São 30 tasks, apresentadas em 3 blocos por tema, e não task por task. O detalhe de cada uma está no [consolidado](consolidado.md).

| Bloco | Tempo |
|-------|-------|
| 0. Abertura | 1 min |
| 1. Login Hipster: qualidade e segurança | 4 min |
| 2. Observabilidade: Login e Identity | 3 min |
| 3. Partner Portal (iPaaS) | 4 min |
| 4. Fechamento | 1 min |

---

## 0. Abertura (1 min)

> "Nesse período foram 30 entregas em 3 frentes. No Login Hipster, o foco foi qualidade e observabilidade. No Partner Portal, foi abrir o Integration Hub pra todos e melhorar o mapeamento com IA. No Identity, foram dois ajustes de back com impacto em produção. Vou passar por tema."

---

## 1. Login Hipster: qualidade e segurança (4 min)

**Mensagem:** o login saiu de quase sem teste pra protegido na CI, e os testes já se pagaram.

- **Cobertura unitária de 18% pra 97,29% de linhas.** A CI agora barra MR que derrube a cobertura abaixo de 80%. Tasks MW-91265 a MW-91269.
- **E2E com Playwright, de zero pra 65 testes.** Cobre login, SSO, troca de conta, código por email, migração, cadastro, convite, senha e páginas de status. Passou em 3 execuções seguidas. Tasks MW-91270 a MW-91272 e MW-91398.
- **Os testes acharam problemas reais:**
  - No cadastro, achei e fechei um caminho que deixava criar conta sem validar o SMS (MW-91266).
  - Os E2E acharam problemas de UX, de acessibilidade e de XSS. Viraram a MW-91677, que fechou duas XSS (`/email` e `/redirect?dest=javascript:`), deixou a troca de conta usável só pelo teclado e no mobile, e faz o leitor de tela anunciar os erros.
- **Sonar zerado** (MW-91959 e MW-92420). Junto veio um teste que quebra a CI se uma chave i18n não tiver tradução.

**Mostrar** (prints da MW-91677):
- [Payload HTML aparece como texto](https://gitlab.miisy.me/identity/meweb/me-login-hipster/uploads/ae2c0a9d059c12e1e0f8132db80a5266/03-email-xss-as-text.png)
- [Troca de conta com foco de teclado](https://gitlab.miisy.me/identity/meweb/me-login-hipster/uploads/06402df5c26916082d72865f11c81a45/01-switch-keyboard-focus.png)

---

## 2. Observabilidade: Login e Identity (3 min)

**Mensagem:** dá pra ver onde o usuário trava no login e por que falha, sem vazar dado pessoal.

- **Eventos de negócio em todas as jornadas do login.** Login, SSO, troca de conta, código por email, cadastro, senha e convite, com tentativa, sucesso e falha com motivo (MW-91273 e MW-91274).
- **Sem dado pessoal na telemetria.** Email, username e tokens saíram, e o trace por usuário é buscado pelo `user.id` (MW-91954).
- **Dashboard do Login Hipster no Grafana,** igual ao do Partner Portal: funis de login e cadastro, falhas por motivo, saúde da API e dos pods e Web Vitals (MW-91275).
  - *Se perguntarem de técnica:* a busca de traces passava dos 30 s e dava "Bad Request". Reescrevi a query, e ela caiu pra 2 a 11 s.
- **Identity:**
  - O webhook de expirar convite gerava **97% dos logs de Error do Identity em prod.** Agora ele ignora o convite que já não está pendente (MW-92454).
  - As métricas de negócio (usuário criado, atualizado etc.) passaram a ser exportadas, e o `/metrics` duplicado saiu (MW-92453).

**Mostrar:** [Dashboard, seção Jornada do Usuário](https://gitlab.miisy.me/observability/dashboards/uploads/9f98125036512d6a6cfcedab0faeb981/ev-1-jornada.png)

---

## 3. Partner Portal (iPaaS) (4 min)

**Mensagem:** o Integration Hub está aberto pra todos os parceiros, e o mapeamento com IA ficou mais previsível.

- **Integration Hub liberado pra todos os tenants** (MW-91984). Saiu a lista de tenants no código, e liberar um parceiro não exige mais deploy.
- **Mapeamento com IA:**
  - A transformação é gerada do zero e com o campo de origem, então o usuário não fica mais preso numa expressão errada. A IA também não usa mais variáveis, que o mapeamento não consegue reescrever (front MW-92534, back MW-92533).
  - Na primeira análise, o campo que só formata texto vira ligação direta, e o mapeamento fica mais limpo (MW-92558).
  - No outbound, mudar o payload não perde mais o campo novo nem quebra as ligações. A barra de etapas não pula mais a validação (MW-92535).
- **Usabilidade:** filtros e ordenação em My Integrations (MW-89268), e os KPIs do mapeamento viraram filtros clicáveis (MW-89272).
- **Qualidade e operação:**
  - E2E das 3 integrações: Inbound, Outbound e Product Search (MW-90827 a MW-90829).
  - Golden Signals e dashboard FinOps do wizard (MW-88298).
  - O log do BFF ficou ligado ao trace (MW-91294).

**Mostrar** (prints da MW-92535): [campo novo aparece e as ligações continuam](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/1b2ab19e93b0b431b4de246c5da7a5bc/03-mapping-item-novo.png). Se der tempo, mostrar também [IA ligando o campo novo](https://gitlab.miisy.me/partner-ecosystem/partner-portal/uploads/33aadfd694fb7c45e71749f460751e75/04-mapping-reanalise-2.png).

---

## 4. Fechamento (1 min)

> "Resumindo: o login tem 97% de cobertura, 65 testes E2E e Sonar zerado, e os testes já fecharam XSS e um bug de cadastro. Ganhou também observabilidade de ponta a ponta sem dado pessoal. O Integration Hub está aberto pra todos, com o mapeamento com IA mais previsível. E o ruído de logs de erro no Identity caiu."

**Pendências pra citar, se perguntarem:**
- A ordenação por última alteração em My Integrations depende do Hub expor as datas (MW-90812).
- Os eventos das jornadas do login só aparecem em prod depois do próximo deploy.
- Depois do deploy do Identity, conferir no Grafana o `me_id_user_created_total` e a participação do `/metrics` no tráfego.

---

## Se faltar tempo, cortar nesta ordem
1. Usabilidade e qualidade/operação do bloco 3 (só citar "E2E das 3 integrações").
2. A explicação técnica do dashboard no bloco 2.
3. O segundo print do bloco 3.
