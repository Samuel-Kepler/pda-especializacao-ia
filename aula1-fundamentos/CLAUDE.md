# aula1-fundamentos

## O que é este projeto

Campo de treino da Aula 1 (Fundamentos) da Especialização em IA da PDA.
Projeto Node pequeno com funções utilitárias em `src/` e testes em `tests/`.

## Stack

- Node.js ≥18 (testado em v20)
- Módulos ES (`"type": "module"` no `package.json`), `import`/`export`
- Runner de testes: `node:test` (nativo do Node, sem dependência externa)

## Comandos

- `npm test` — roda a suíte inteira uma vez
- `npm run test:watch` — roda em modo watch durante o desenvolvimento

## Regras

- Os arquivos em `tests/` são a especificação. **Nunca edite testes** pra fazê-los passar.
- Antes de dizer que uma tarefa terminou, rode `npm test` e mostre a saída. "Terminei" sem os testes verdes não vale.
- Estilo: `const`/`let` (nunca `var`), sempre `===`/`!==`, mensagens de erro e comentários em português.

## Como eu quero trabalhar com você

- Antes de editar, explique em 1–2 frases o que vai mudar e por quê.
- Mudanças pequenas e focadas. Uma tarefa por vez.
- Se não tiver certeza sobre uma API ou lib, consulte a documentação (MCP context7) em vez de chutar.
- Se for chutar algo (nome de método, comportamento de lib), avise explicitamente em vez de apresentar como certeza.
