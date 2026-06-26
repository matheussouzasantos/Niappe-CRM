# `.claude/context` — Base de conhecimento do agente

Este diretório é o **único lugar** onde o Claude Code (e qualquer outro agente
de IA) deve consultar contexto curado deste projeto: boas práticas de código,
identidade de marca, contexto de produto/domínio e qualquer referência
duradoura que deva guiar o trabalho.

> **Não é auto-carregado.** Estes arquivos **não** são injetados no contexto a
> cada mensagem (isso desperdiçaria tokens). O `AGENTS.md` na raiz aponta para
> cá com *gatilhos* — "antes de fazer X, leia Y". Abra o arquivo relevante
> **sob demanda**, quando a tarefa pedir. Esse é o padrão de _progressive
> disclosure_: o índice é barato; o conteúdo completo só entra quando importa.

## Mapa — quando ler o quê

| Quando você está...                                                         | Leia                                          |
| --------------------------------------------------------------------------- | --------------------------------------------- |
| Escrevendo, refatorando ou revisando **qualquer** código                    | [`engineering/codigo-limpo.md`](engineering/codigo-limpo.md)         |
| Projetando módulos, fronteiras, dependências, estrutura de pastas           | [`engineering/arquitetura-limpa.md`](engineering/arquitetura-limpa.md) |
| Mexendo em UI, naming visível ao usuário, cores, copy, identidade visual    | [`brand/README.md`](brand/README.md)          |
| Precisando de contexto de produto / domínio / regras de negócio             | [`product/README.md`](product/README.md)      |

## O que vive aqui

- **`engineering/`** — Boas práticas de código, prescritivas e independentes de
  linguagem. Aplicáveis a todo o código do repositório.
  - `codigo-limpo.md` — Clean Code (nível micro: nomes, funções, classes,
    comentários, testes, _smells_). Tem uma checklist de pré-commit no fim.
  - `arquitetura-limpa.md` — Clean Architecture (nível macro: SOLID,
    componentes, Regra da Dependência, fronteiras, política vs. detalhe).
- **`brand/`** — Identidade visual e de comunicação (cores, tipografia, tom de
  voz, logos). Preencha o template antes de gerar UI ou textos de marca.
- **`product/`** — Contexto de produto e domínio que **não** dá para deduzir só
  lendo o código (objetivos, personas, vocabulário do domínio, decisões e
  restrições). Preencha conforme o projeto evolui.

## Como adicionar contexto novo (convenções)

Mantenha esta base limpa — as próprias regras de `engineering/codigo-limpo.md`
valem aqui (Regra do Escoteiro inclusa):

1. **Um assunto por arquivo.** Nome em `kebab-case`, descritivo
   (`tom-de-voz.md`, não `notas.md`).
2. **Coloque no bucket certo** (`engineering/`, `brand/`, `product/`) ou crie um
   bucket novo se nenhum servir — e então registre-o no **Mapa** acima.
3. **Registre todo arquivo novo** no Mapa e/ou na seção "O que vive aqui", com
   uma linha dizendo *quando* lê-lo. Um arquivo que ninguém sabe quando abrir é
   contexto morto.
4. **Adicione o gatilho no `AGENTS.md` da raiz** se o arquivo deve ser
   consultado proativamente durante o trabalho (ex.: boas práticas de código).
5. **Prefira referência a `@import`.** Não puxe arquivos grandes para dentro do
   `CLAUDE.md`/`AGENTS.md` via `@`; isso os carrega em todo contexto. Aponte o
   caminho e deixe o agente abrir sob demanda.
6. **Mantenha atualizado.** Documento desatualizado engana mais do que ajuda —
   corrija ou apague.
