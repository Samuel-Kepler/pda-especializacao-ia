# REVIEW — Aula 1

## O que eu pedi

**Desafio mínima — `validaCpf`:**
> Corrija `src/validaCpf.js` para rejeitar CPFs com todos os dígitos iguais (ex.: `111.111.111-11`) e
> strings com letras no meio. `npm test` tem que passar. Não edite os arquivos em `tests/`.

**Desafio 3 — `fetchUsuario`:**
> Os testes em `tests/fetchUsuario.test.js` estão falhando. Sem editar `tests/`, ajuste
> `src/fetchUsuario.js` pra: (1) lançar um erro com o status HTTP no texto quando a API responde
> um status de erro (ex.: 404); (2) não quebrar quando a API devolve `nome`/`email` faltando —
> deve cair pra string vazia; (3) rejeitar `id` inválido (não-inteiro ou ≤ 0) **antes** de chamar
> `fetchFn`, com uma mensagem de erro que mencione "id". `npm test` verde ao final.

## O que o agente fez

**`validaCpf.js`** — leu `tests/validaCpf.test.js` primeiro (é a especificação), identificou os
dois testes que faltavam passar e adicionou duas guardas *antes* do cálculo do dígito verificador:

```diff
+  if (/[a-zA-Z]/.test(cpf)) return false;
   ...
+  if (/^(\d)\1{10}$/.test(digitos)) return false;
```

Também trocou o `==` solto na comparação final por `===` (regra de estilo que ficou no `CLAUDE.md`).
1 volta de loop: leu o teste → editou → rodou `npm test` → passou de primeira.

**`fetchUsuario.js`** — leu `tests/fetchUsuario.test.js`, viu os 3 `assert.rejects` que faltavam e
os campos que precisavam de fallback. Adicionou validação de `id` no topo da função (retorna antes
de chamar `fetchFn`, como o teste exige — "não pode ter chamado a API"), checagem de `resposta.ok`
com o status no texto do erro, e `?? ''` nos campos que podem vir `undefined`.
1 volta de loop também: `npm test` passou na primeira tentativa depois da edição.

**Resultado:** `npm test` → 13/13 passando (antes: 8/13).

## O que eu aceitei e por quê

- As duas guardas em `validaCpf`: resolvem exatamente o que os testes pedem, sem tocar no cálculo
  do dígito verificador que já funcionava.
- A ordem da validação de `id` em `fetchUsuario` (antes do `fetchFn`): é o que o teste
  `rejeita id inválido antes de chamar a API` exige — o mock marca `chamou = true` se a API for
  chamada, e o teste falharia se a validação viesse depois.
- `?? ''` em vez de checar cada campo com `if`: mais curto, mesmo comportamento pros casos testados.

## O que eu rejeitei ou mudei e por quê

- Nada precisou ser revertido desta vez — as duas primeiras tentativas já bateram com o teste.
  (Isso é raro o suficiente pra anotar: normalmente rola pelo menos uma volta extra de "quase
  passou". Ver seção de não-determinismo no parágrafo pessoal.)
- Pedi explicitamente pra não editar `tests/` nas duas tarefas — o agente respeitou em ambos os
  casos, não tentou "ajustar a especificação" pra facilitar.

## Onde ele chutou / alucinou / fez mais do que pedi

- Em `validaCpf.js`, ele trocou `dv1 == nums[9]` por `dv1 === nums[9]` sem eu ter pedido — não é
  bug (o teste não cobria isso), mas é escopo a mais: eu só tinha pedido pra rejeitar os dois casos
  novos. Aceitei porque bate com a regra de estilo que **eu mesmo** coloquei no `CLAUDE.md`
  ("sempre `===`/`!==`") — mas percebi que isso é exatamente o tipo de "mudança extra" que eu
  deveria olhar com mais desconfiança se não tivesse essa regra escrita.
- Não chutou nenhuma API ou lib nova — as duas correções usam só `RegExp`/`Number.isInteger`
  nativos do JS, então não teve chance de alucinar assinatura de método.

## O que eu colocaria no CLAUDE.md pra isso não acontecer de novo

- Já coloquei: "se for chutar algo, avise explicitamente em vez de apresentar como certeza" —
  cobre o caso do `===` acima; da próxima vez ele deve dizer "troquei `==` por `===` porque não
  foi pedido, ok?" em vez de só fazer.
- Adicionaria: "mudanças fora do escopo pedido (mesmo que pequenas e corretas) devem vir separadas
  do diff principal, num commit ou num aviso à parte" — pra eu conseguir revisar o que é a correção
  em si e o que é "melhoria de brinde" sem misturar os dois no mesmo `git diff`.

---

## Desafio 2 — skill `gerar-commit`

Criei `.claude/skills/gerar-commit/SKILL.md`: gera mensagem de commit a partir do `git diff --staged`,
no padrão do repo (Conventional Commits em português), e **recusa a sugerir uma mensagem única se o
diff staged misturar mudanças não relacionadas**.

### Exemplo de invocação

Depois de dar `git add` em `CLAUDE.md`, `src/validaCpf.js`, `src/fetchUsuario.js` e no próprio
`SKILL.md` de uma vez, rodei `/gerar-commit`. Resultado (a skill seguiu o próprio passo 3 — apontou
a mistura antes de sugerir texto):

```
Esse diff staged mistura 3 mudanças sem relação direta: (1) preenchimento de TODOs no CLAUDE.md,
(2) fix em validaCpf.js, (3) fix em fetchUsuario.js, (4) nova skill gerar-commit. Sugiro separar em
commits menores. Se preferir um commit só mesmo assim, aqui vai uma mensagem que cobre o conjunto:

docs: completa CLAUDE.md e corrige validaCpf/fetchUsuario com skill de commit

- CLAUDE.md: remove TODOs (stack, comandos, regras de estilo)
- validaCpf: rejeita dígitos repetidos e strings com letras
- fetchUsuario: valida id, propaga status HTTP no erro, tolera campos faltando
- adiciona skill .claude/skills/gerar-commit
```

Rodei de novo só com `src/fetchUsuario.js` staged (isolado), pra comparar:

```
fix: fetchUsuario valida id e trata erro HTTP e campos ausentes

Sem essas guardas, um id inválido chegava até a API e uma resposta 4xx/5xx virava
TypeError ao tentar ler campos que não existem.
```

A segunda é o formato que eu de fato uso — confirma que vale a pena comitar em pedaços menores,
que foi exatamente o que segui abaixo.

---

## Pendente (precisa ser feito por mim, fora do Claude Code)

- [ ] Print do `/mcp` (context7 conectado) — `.mcp.json` já configurado localmente
      (`claude mcp add context7 -s project -- npx -y @upstash/context7-mcp`), falta abrir o
      Claude Code interativo e tirar o print.
- [ ] Print do `/status`
- [ ] Parágrafo (5–8 linhas) sobre onde apareceu *contexto*, *temperatura/não-determinismo* ou
      *alucinação* na minha sessão de hoje (labs 1 e 2 do `GUIA-DO-ALUNO.md`)
- [ ] Bônus (opcional): rodar o desafio 3 duas vezes em sessões novas e comparar os diffs
