---
name: gerar-commit
description: Gera uma mensagem de commit a partir do diff staged, no padrão Conventional Commits em português. Use quando o usuário pedir pra escrever, sugerir ou gerar uma mensagem de commit, ou invocar /gerar-commit.
allowed-tools: Bash(git diff:*), Bash(git status:*), Bash(git log:*)
---

# Gerar commit

Você escreve a mensagem de commit para o diff que já está staged (`git add`). O objetivo não é
descrever o diff linha a linha — é explicar a intenção da mudança pra quem vai ler o `git log` depois.

## Entrada

Sem `$ARGUMENTS`: use o diff staged atual. Se vier algo em `$ARGUMENTS`, trate como contexto extra
que o autor quer que você leve em conta (ex.: número da issue, motivo da mudança).

## Passos

1. Rode `git status` e `git diff --staged`. Se não houver nada staged, avise e pare — não gere
   mensagem para um diff vazio nem para o working tree inteiro sem estar staged.
2. Rode `git log -5 --oneline` pra pegar o padrão de mensagens já usado neste repo (tom, idioma,
   se usa prefixo tipo `feat:`/`fix:`).
3. Identifique o "porquê" da mudança a partir do diff — não só o "o quê". Se o diff for puramente
   mecânico (rename, formatação), diga isso.

## Formato da saída

```
<tipo>: <resumo em até 60 caracteres, no imperativo, em português>

<corpo opcional: 1-3 linhas explicando o porquê, só se agregar>
```

Tipos aceitos: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`.

## Regras

- Não rode `git commit` nem `git add` — só sugira a mensagem. Quem decide comitar é o autor.
- Não invente motivação que não está no diff. Se não der pra saber o porquê, descreva só o quê.
- Uma mensagem por vez. Se o diff staged mistura mudanças não relacionadas, aponte isso antes de
  sugerir a mensagem — commits misturados são um problema, não algo pra mascarar com uma boa mensagem.
