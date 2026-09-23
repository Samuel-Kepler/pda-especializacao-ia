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

## LAB 1 — Tokenização (platform.openai.com/tokenizer)

| item | tokens | caracteres | observação |
|---|---|---|---|
| `Programadores do Amanhã` | 5 | 23 | "Programadores" quebra em "Program" + "adores" |
| Parágrafo em português | 20 | 85 | |
| Mesmo parágrafo em inglês | 17 | 87 | **menos tokens que o PT**, mesmo com mais caracteres |
| `src/validaCpf.js` inteiro | 277 | 842 | código é ~0,33 token/caractere vs. ~0,2–0,24 da prosa — bem mais denso |
| `1234567890` | 4 | 10 | |
| `1 2 3 4 5 6 7 8 9 0` | 19 | 19 | espaçar os dígitos quase quintuplicou o custo em token |

**O que custa mais token — português ou inglês? código ou prosa?** Português custou mais que
inglês (20 vs. 17, mesmo o inglês tendo *mais* caracteres) — acentos e a morfologia do português
quebram em mais pedaços no vocabulário do tokenizer, que foi treinado majoritariamente em inglês.
Código custou muito mais que prosa por caractere: símbolos, indentação e nomes de variável em
`camelCase` não colam com os tokens comuns do vocabulário do jeito que palavras inteiras colam.

## LAB 2 — Temperatura (Google AI Studio, modelo `gemma-4-26b-a4b-it`)

Prompt fixo: `Escreva uma função JavaScript validaCpf(cpf) que retorna true ou false. Só o código, sem explicação.`
Cada rodada foi numa sessão nova (sem contexto da rodada anterior).

**T = 0, 5 rodadas** — a lógica central ficou quase igual em 4 das 5:

| rodada | o que mudou vs. a anterior |
|--------|------|
| 1 | baseline: `base[i]`, peso `(base.length + 1 - i)`, acesso `cpf[9]` |
| 2 | só troca de sintaxe: `base.charAt(i)` / `cpf.charAt(9)` em vez de colchetes |
| 3 | **mudança funcional**: peso virou `(base.length - i)`, sem o `+1` — quebra o cálculo do dígito |
| 4 | igual à rodada 1 |
| 5 | igual à rodada 1 |

**T = 2 (máximo), 5 rodadas** — muito mais variação, inclusive degeneração:

| rodada | o que mudou vs. a anterior |
|--------|------|
| 1 | abordagem diferente da família toda: loops `1..9`/`1..10`, pesos `(11-i)`/`(12-i)`, e **citou uma fonte externa** (`mco2.com.br`) mesmo com "Grounding with Google Search" desligado |
| 2 | **resposta vazia** (só um `.`) com uma citação pra um repositório GitHub aleatório — degenerou |
| 3 | igual à rodada 1 (mesma abordagem, mesma citação) |
| 4 | terceira abordagem: peso decrescente (`peso--`) e fórmula do resto invertida (`11 - resto`) |
| 5 | variação da abordagem original, mas com **bug de precedência de operador**: `parseInt(base[i]) * base.length - i` sem parênteses — não é `* (base.length - i)`, é `(... * base.length) - i` |

**Se o mesmo prompt dá respostas diferentes, o que isso muda em como eu testo código gerado por IA?**
Em T=0 ainda apareceu uma variação funcional (rodada 3) — ou seja, mesmo "determinístico" não é
garantia. Em T=2 apareceu uma resposta vazia e um bug de precedência que um `git diff` rápido não
pega de olho (parece código correto até você rodar o teste). Isso é o argumento inteiro pra nunca
aceitar código de IA sem rodar os testes: a mesma pergunta, no mesmo minuto, pode gerar a versão
certa e a versão quebrada.

## Bônus — alucinação vs. context7 (`@pda/valida-cpf`, lib que não existe)

Perguntei sem pedir o context7 explicitamente: `como instalo e uso a lib @pda/valida-cpf?`
O agente **não alucinou** — foi direto checar `npm view @pda/valida-cpf` (deu 404) antes de
responder, e concluiu que a lib não existe, sugerindo usar o `validaCpf` local do projeto.
Perguntei de novo pedindo `usando o context7` e ele confirmou pela mesma via (resolve-library-id
não achou nada). Resultado honesto: diferente do que o guia da aula esperava (o clássico
"ele inventa o pacote"), aqui o `CLAUDE.md` ("se não tiver certeza, consulte a documentação em vez
de chutar") já bastou pra evitar a alucinação antes mesmo do context7 entrar em ação — o context7
só confirmou o que o `npm view` já tinha mostrado.

## Parágrafo — contexto / temperatura / alucinação (rascunho, ajustar pra minhas palavras)

> O LAB 2 deixou escancarada a relação entre temperatura e não-determinismo. Rodando o mesmo prompt cinco vezes em $T=0$, eu esperava cinco respostas idênticas. Só que na terceira rodada o modelo mudou a fórmula do peso e quebrou a lógica toda — o que prova que determinístico não significa correto. Já em $T=2$, o cenário descarrilou: foram três abordagens completamente diferentes pro mesmo problema, uma resposta em branco e um bug de precedência de operador que eu só peguei executando o teste, porque lendo o código passava batido.Na parte de contexto, a diferença ficou clara quando comparei pedir uma biblioteca inexistente com e sem o context7. No meu setup, como o CLAUDE.md já instruía o agente a validar informações antes de responder, eu nem cheguei a ver a alucinação clássica do pacote inventado — o próprio agente tomou a iniciativa de rodar um npm view para checar. Isso me provou que a melhor defesa contra alucinação não é depender só de uma ferramenta isolada: precisa de instrução bem alinhada no prompt/contexto e de testes automatizados rodando de verdade, até porque a exata mesma pergunta pode dar certo agora e quebrar na próxima execução.

---

