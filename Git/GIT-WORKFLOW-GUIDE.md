# Git Workflow - Trabalhando em múltiplas tarefas simultaneamente

## O problema

Você tem um MR aberto em review e precisa começar outra tarefa no mesmo projeto.
Se você fizer as mudanças na branch errada e depois tentar separar, vai gerar conflitos.

---

## Cenário 1: Tarefa 2 NÃO depende da tarefa 1

A tarefa nova não precisa do código que está em review.

```bash
# Vá para develop/main e atualize
git checkout develop
git pull

# Crie a branch nova a partir de develop
git checkout -b feature/MW-XXXXX
```

Trabalhe normalmente. Sem conflitos, sem complicação.

---

## Cenário 2: Tarefa 2 DEPENDE da tarefa 1

A tarefa nova precisa do código que está em review.

```bash
# Crie a branch nova a partir da branch da tarefa 1
git checkout feature/tarefa-1
git checkout -b feature/MW-XXXXX
```

Trabalhe normalmente. Quando a tarefa 1 for mergeada no develop:

```bash
git checkout feature/MW-XXXXX
git fetch origin
git rebase origin/develop
```

O rebase vai replantar seus commits em cima do develop atualizado (que já inclui a tarefa 1).
O MR da tarefa 2 vai mostrar somente os commits dela.

---

## Cenário 3 (emergência): Já fez alterações na branch errada, MAS NÃO commitou

Você fez mudanças na branch da tarefa 1 (ou em develop) sem commitar.

```bash
# 1. Guarde tudo no stash (incluindo arquivos novos com -u)
git stash push -u -m "alteracoes-tarefa-2"

# 2. Vá para develop e atualize
git checkout develop
git pull

# 3. Crie a branch nova
git checkout -b feature/MW-XXXXX

# 4. Traga as mudanças de volta
git stash pop
```

Se der conflito no stash pop (ex: package.json, yarn.lock):

```bash
# Resolva os conflitos no editor (remova os marcadores <<<<<<< ======= >>>>>>>)
# Para yarn.lock, aceite a versão do develop e regenere:
git checkout --theirs yarn.lock
yarn install

# Depois marque como resolvido
git add .
```

---

## Cenário 4 (emergência): Já commitou tudo misturado na mesma branch

Você commitou mudanças da tarefa 2 na branch da tarefa 1.

```bash
# 1. Identifique os commits da tarefa 2
git log --oneline feature/tarefa-1

# 2. Crie a branch nova a partir de develop
git checkout develop
git pull
git checkout -b feature/MW-XXXXX

# 3. Traga somente os commits da tarefa 2 com cherry-pick
git cherry-pick <hash1> <hash2> <hash3>

# 4. Volte para a branch da tarefa 1 e remova os commits da tarefa 2
git checkout feature/tarefa-1
git rebase -i origin/develop
# No editor, delete as linhas dos commits da tarefa 2 e salve
```

---

## Resumo rápido

| Situação | Comando chave |
|---|---|
| Começar tarefa nova (sem dependência) | `git checkout develop && git checkout -b feature/nova` |
| Começar tarefa nova (com dependência) | `git checkout feature/tarefa-1 && git checkout -b feature/nova` |
| Mudanças não commitadas na branch errada | `git stash push -u` → trocar branch → `git stash pop` |
| Commits misturados na branch errada | `git cherry-pick` para trazer commits específicos |
| Conflito em yarn.lock / package-lock.json | `git checkout --theirs yarn.lock && yarn install` |

## Regra de ouro

**Sempre crie uma branch separada ANTES de começar a trabalhar em algo novo.**
Nunca continue commitando na branch que está em review.
