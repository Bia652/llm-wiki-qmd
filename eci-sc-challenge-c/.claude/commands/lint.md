---
description: Valida front matter, links e coerência do schema (incl. status/confidence-score/supersedes/last-verified) em todas as páginas da wiki
---

## Contexto

Vais validar todas as páginas `.qmd` da wiki real em `wiki/` (ver `CLAUDE.md`). Nunca uses
`_proto/` — é só evidência de protótipo e fixtures de teste. Lê cada ficheiro
`.qmd`, não os `.llms.md` renderizados (o lint valida a fonte, não o output do Quarto). Reporta
todos os problemas encontrados, agrupados por tipo, com o caminho do ficheiro e a linha/campo em
causa. Não corrijas automaticamente — reporta, e pergunta antes de editar.

Páginas de listagem/índice (`index.qmd` com bloco `listing:` no front matter, sem `categories`/
`related`) estão fora do âmbito das validações de schema de conteúdo abaixo — só entram nas
verificações de links partidos.

## 1. Front matter obrigatório (páginas de conteúdo: `type: document`, `type: concept`, `type: glossary`)

Campos sempre obrigatórios: `title`, `type`, `categories`, `last-ingested`.
Documentos (`type: document`) têm ainda: `source-pdf`, `source-version`.

## 2. Schema v2 — `status` / `confidence-score` / `supersedes` / `last-verified`

- **`status`**: se presente, tem de ser um destes 4 valores exatos: `draft`, `verified`,
  `stale`, `deprecated`. Qualquer outro valor (erro de escrita, capitalização diferente, etc.) é
  um erro de lint.
- **`confidence-score`**: se presente, tem de ser um número entre `0.0` e `1.0` inclusive.
  Fora deste intervalo, ou não-numérico, é erro.
- **`supersedes`**: se presente, tem de ser uma lista (`[]` se vazia). Cada entrada tem de ser um
  caminho para outra página `.qmd` que **exista de facto** na wiki — reporta como erro qualquer
  `supersedes` a apontar para um ficheiro inexistente (referência partida).
- **`last-verified`**: se presente, tem de ser uma data válida (`YYYY-MM-DD`) e **não pode ser
  anterior a `last-ingested`** — verificar algo antes de o ingerir não faz sentido; reporta como
  aviso (não erro bloqueante) se isso acontecer.
- Se uma página usa **qualquer um** destes 4 campos, é boa prática (aviso, não erro) usar os 4 —
  meias-medidas (só `status`, sem `last-verified`, por exemplo) tornam o schema menos útil para o
  `/query`.

## 3. Coerência semântica entre páginas (o que os testes de A/B mostraram ser crítico)

- **Toda a página com `status: stale` ou `status: deprecated` tem de ser referenciada pelo
  `supersedes` de pelo menos uma outra página.** Uma página `stale` "órfã" (sem ninguém a
  substituí-la explicitamente) é um sinal sem utilidade para o `/query` — ou falta escrever a
  página substituta, ou o `status` está errado.
- **Duas páginas não podem afirmar factos diferentes sobre o mesmo tópico sem uma relação
  `supersedes` entre elas.** Isto é heurístico, não 100% automatizável por regex — usa o teu
  julgamento: se duas páginas partilham `categories`/`related` e parecem tratar do mesmo dado
  concreto (ex.: o mesmo prazo, o mesmo valor técnico) com valores diferentes, e nenhuma faz
  `supersedes` da outra, reporta como aviso de possível deriva semântica/conflito não
  documentado. (Este cenário foi reproduzido deliberadamente no teste A/B do schema v2 — sem
  `supersedes` a ligar as duas páginas, o agente de `/query` fica sem forma fiável de saber qual
  delas é atual.)
- **Duas páginas `type: concept` não devem afirmar substancialmente os mesmos factos, sem
  conflito nenhum entre elas, e sem que uma cite/estenda a outra.** Isto é o sinal inverso do
  ponto anterior: não é deriva semântica (os factos não diferem), é duplicação — normalmente
  produto de duas cristalizações do `/query` (`query.md`, Passo 5) que deviam ter atualizado a
  mesma página em vez de criar páginas paralelas. Heurística: `categories`/`related` sobrepostos
  + corpo do texto a cobrir o mesmo cruzamento de documentos, sem nenhuma referenciar a outra em
  `related` nem em `supersedes`. Reporta como aviso de possível página redundante a fundir; não
  fundas automaticamente.

## 4. Links e referências

- Todos os links relativos entre páginas `.qmd` (`[texto](caminho.qmd)`, incluindo os que
  aparecem dentro de `related:`) têm de apontar para ficheiros que existem.
- Toda a página `type: document` tem de ter um link de volta ao PDF/artifact original
  (`source-pdf` ou um link na secção "Fonte") — páginas sem origem rastreável são erro.

## 5. Relatório final

Resume no fim: nº de ficheiros verificados, nº de erros (bloqueantes: front matter inválido,
`supersedes`/link partido, `status` fora do enum, `confidence-score` fora de 0–1) vs. nº de
avisos (heurísticos: possível deriva semântica, `status: stale` órfão, schema parcialmente
aplicado, `last-verified` antes de `last-ingested`). Não avances para nenhuma correção sem
confirmação.
