# Brag document: ME

## Fluxo

1. Concluiu a task (MR aprovado e validado no ambiente)? Cria `tasks/YYYY-MM-DD_MW-XXXXX_slug/README.md` a partir de `_templates/task.md`. Imagens na mesma pasta.
2. Chegou a review? Cria `reviews/YYYY-MM-DD/consolidado.md` a partir de `_templates/consolidado.md`, listando as tasks com `data` depois da última review, agrupadas por projeto.
3. Apresenta o consolidado. Esqueceu algum detalhe? Abre o README da task.

## Estrutura

| Pasta | Conteúdo |
|-------|----------|
| `tasks/` | Uma pasta por task concluída (descrição completa + imagens) |
| `reviews/` | Uma pasta por review (consolidado apresentado) |
| `_templates/` | Moldes de task e consolidado |

Datas sempre `YYYY-MM-DD` para ordenar certo.
