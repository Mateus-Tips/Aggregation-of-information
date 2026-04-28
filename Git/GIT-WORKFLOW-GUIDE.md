# Git Workflow - Guia rápido para tarefas simultâneas

Este guia foi organizado para consulta rápida no dia a dia:
- Comece pelo **"Comandos mais usados"**.
- Use os **cenários** quando acontecer algo fora do fluxo ideal.

---

## Comandos mais usados (cola rápida)

### 1) Começar tarefa nova

#### Sem dependência da tarefa em review
```bash
git checkout develop
git pull
git checkout -b feature/MW-XXXXX
```

#### Com dependência da tarefa em review
```bash
git checkout feature/tarefa-1
git checkout -b feature/MW-XXXXX
```

Quando a tarefa 1 for mergeada em `develop`:
```bash
git checkout feature/MW-XXXXX
git fetch origin
git rebase origin/develop
git push --force-with-lease
```

---

### 2) Git stash (guardar e recuperar alterações)

#### Stash de tudo (arquivos modificados + novos)
```bash
git stash push -u -m "mensagem-do-stash"
```

#### Listar stashes
```bash
git stash list
```

#### Ver o conteúdo de um stash
```bash
git stash show -p stash@{0}
```

#### Aplicar stash sem remover da pilha
```bash
git stash apply stash@{0}
```

#### Aplicar stash e remover da pilha
```bash
git stash pop
```

#### Apagar um stash específico
```bash
git stash drop stash@{0}
```

#### Apagar todos os stashes
```bash
git stash clear
```

---

### 3) Branches (listar, apagar e limpar locais órfãs)

#### Listar branches locais
```bash
git branch
```

#### Listar branches remotas
```bash
git branch -r
```

#### Criar branch nova
```bash
git checkout -b feature/MW-XXXXX
```

#### Trocar de branch
```bash
git checkout nome-da-branch
```

#### Apagar branch local (somente se já estiver mergeada)
```bash
git branch -d nome-da-branch
```

#### Forçar apagar branch local (cuidado)
```bash
git branch -D nome-da-branch
```

#### Apagar branch remota
```bash
git push origin --delete nome-da-branch
```

#### Atualizar referências remotas e remover remotos já deletados
```bash
git fetch --prune
```

#### Apagar branches locais cujo remoto não existe mais
```bash
git fetch --prune
git branch -vv | grep ': gone]' | awk '{print $1}' | xargs -r git branch -d
```

Se alguma não estiver mergeada e você realmente quiser remover:
```bash
git branch -vv | grep ': gone]' | awk '{print $1}' | xargs -r git branch -D
```

---

## Cenários detalhados

### Cenário 1: Tarefa 2 NÃO depende da tarefa 1

A tarefa nova não precisa do código que está em review.

```bash
git checkout develop
git pull
git checkout -b feature/MW-XXXXX
```

Trabalhe normalmente. Sem conflitos, sem complicação.

---

### Cenário 2: Tarefa 2 DEPENDE da tarefa 1

A tarefa nova precisa do código que está em review.

```bash
git checkout feature/tarefa-1
git checkout -b feature/MW-XXXXX
```

Quando a tarefa 1 for mergeada no `develop`:

```bash
git checkout feature/MW-XXXXX
git fetch origin
git rebase origin/develop
git push --force-with-lease
```

O rebase vai replantar seus commits em cima do `develop` atualizado (que já inclui a tarefa 1).
O MR da tarefa 2 vai mostrar somente os commits dela.

---

### Cenário 3 (emergência): Fez alterações na branch errada, MAS NÃO commitou

Você fez mudanças na branch da tarefa 1 (ou em `develop`) sem commitar.

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

Se der conflito no `stash pop` (ex: `package.json`, `yarn.lock`):

```bash
# Resolva os conflitos no editor (remova os marcadores <<<<<<< ======= >>>>>>>)
# Para yarn.lock, aceite a versão do develop e regenere:
git checkout --theirs yarn.lock
yarn install

# Depois marque como resolvido
git add .
```

---

### Cenário 4 (emergência): Já commitou tudo misturado na mesma branch

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

## Resumo em 30 segundos

| Situação | Comando chave |
|---|---|
| Começar tarefa nova (sem dependência) | `git checkout develop && git checkout -b feature/nova` |
| Começar tarefa nova (com dependência) | `git checkout feature/tarefa-1 && git checkout -b feature/nova` |
| Mudanças não commitadas na branch errada | `git stash push -u -m "msg"` -> trocar branch -> `git stash pop` |
| Commits misturados na branch errada | `git cherry-pick` para trazer commits específicos |
| Limpar locais órfãs do remoto | `git fetch --prune` + remover branches `: gone]` |

## Regra de ouro

**Sempre crie uma branch separada ANTES de começar algo novo.**
Nunca continue commitando na branch que está em review.
