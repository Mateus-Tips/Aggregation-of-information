---
data: 2026-10-05
ticket: MW-89270
ticket_url: https://meagil.atlassian.net/browse/MW-89270
mr:
  - https://gitlab.miisy.me/partner-ecosystem/partner-portal/-/merge_requests/500
projeto: Partner Portal (iPaaS)
tipo: melhoria-ux
camada: [front]
---

# Ctrl+Click / botão do meio abre em nova aba

## Resumo
Os botões de navegação do Partner agora abrem em nova aba com Ctrl/Cmd+Click e botão do meio. O clique simples continua na mesma aba, sem recarregar.

## Problema
Os botões eram `click` + `router.push`, então o browser não tinha link pra abrir em nova aba.

## Por que
Usabilidade básica: comparar integrações, abrir execuções em paralelo.

## Como
- Os botões viraram links reais (`<a href>`, `router-link custom`, `nuxt-link`) e o browser cuida da nova aba.
- O header do me-toolkit chama `item.click()` sem o evento: o handler lê `globalThis.event` e, com modificador ou botão do meio, deixa o browser agir. No clique simples faz `preventDefault` + `router.push`.
- Cobertura: header, side menu, cards de Templates e My Integrations, "View timeline", drawer do Dashboard, conectores.

## Onde
partner-portal: `topNavItems`, `IntegrationCard`, páginas de integrações, execuções, Dashboard, conectores

## Resultado / impacto
Comportamento padrão de web em toda a navegação do Partner. O drawer do Dashboard só fecha quando a navegação é na mesma aba.

## Aprendizado
Workaround com `globalThis.event` pra lib (me-toolkit) que não repassa o evento de clique.

## Imagens
Sem prints no MR.
