Apresentei com os slides (`slides.html`), seguindo o `roteiro.md`. Sonar e Grafana mostrei ao vivo.

Projeto Login Hipster -> Front, BFF e Infra
1- testes: cobertura de 18% pra 97%, 65 testes E2E com Playwright e trava de 80% na CI
2- o que os testes acharam: cadastro sem validar SMS, duas XSS e acessibilidade na troca de conta
3- Sonar zerado e teste de chave i18n sem tradução (ao vivo)
4- observabilidade das jornadas sem dado pessoal e dashboard no Grafana (ao vivo)

Projeto ME Identity -> Back
1- webhook de expirar convite que gerava 97% dos logs de Error em prod
2- métricas de negócio exportadas e /metrics duplicado removido

Projeto Partner Portal (iPaaS) -> Front e BFF e IntegrationHub -> Back
1- Integration Hub liberado pra todos os tenants
2- mapeamento com IA: transformação do zero com campo de origem, ligação direta e realinhamento do outbound quando o payload muda
3- também: filtros em My Integrations, KPIs clicáveis, E2E das 3 integrações, Golden Signals/FinOps e log do BFF ligado ao trace
