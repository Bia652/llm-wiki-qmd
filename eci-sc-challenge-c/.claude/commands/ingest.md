---
description: Ingere um PDF de pdfs/ e gera/atualiza páginas .qmd na wiki, seguindo a Estrutura A e o schema completo (incl. schema v2)
argument-hint: [caminho do PDF em pdfs/]
---

## Contexto

Vais ingerir um PDF novo (ou uma versão nova de um PDF já ingerido) para a wiki, seguindo a
estrutura e o schema definidos em `CLAUDE.md` (Estrutura A: `documents/` + `concepts/` +
`glossary.qmd`, schema de front matter completo incl. `status`/`confidence-score`/`supersedes`/
`last-verified`). A wiki real é `wiki/` — nunca escrever em `_proto/` (evidência de protótipo).

**Nunca alteres nem duplique-se o PDF original em `pdfs/`** — é a fonte imutável. Não inventes
conteúdo que não esteja no PDF; se uma secção for ambígua ou ilegível, di-lo em vez de preencher
com suposições.

## Passo 1 — Converter PDF → Markdown bruto

Usa `markitdown` (`python -m markitdown <pdf> -o _proto/raw-md/<nome>.md`, ver
`requirements.txt`) para obter uma extração de texto bruta. Este ficheiro é matéria-prima
temporária, não faz parte da wiki final.

## Passo 2 — Decidir granularidade: documento novo, extensão de conceito, ou substituição

- **PDF novo, sem sobreposição com conceitos existentes:** cria só `documents/<nome>.qmd`.
- **PDF novo que toca um conceito já existente em `concepts/`:** cria `documents/<nome>.qmd` E
  atualiza a página `concepts/*.qmd` relevante, acrescentando a nova síntese (não duplicar a
  definição do conceito — a página de conceito é a fonte canónica única, ver `CLAUDE.md`, "Regra
  de crescimento": se já agregar >4 fontes, considera subdividir).
- **PDF novo que introduz um conceito ainda sem página de síntese:** cria também
  `concepts/<conceito>.qmd`, com `related` a apontar para os `documents/*.qmd` relevantes.
- **PDF que é uma versão revista de um documento já ingerido** (ex.: nova edição da mesma
  especificação): cria a página nova em `documents/` e marca na página antiga `status: stale` (ou
  `deprecated`); a página nova declara `supersedes: [caminho da página antiga]`. Nunca apagues a
  página antiga — o histórico de versões faz parte da wiki.

## Passo 3 — Escrever o `.qmd` com o schema completo

Front matter obrigatório (ver `CLAUDE.md` para a tabela completa):

```yaml
title: "..."
type: document   # ou concept / glossary
source-pdf: "../../../pdfs/<ficheiro>.pdf"   # só documentos
source-version: "..."                          # só documentos
categories: [...]
related: [...]                                 # se aplicável
last-ingested: <data de hoje>
status: draft            # nova ingestão começa "draft" até ser revista; usar "verified" só
                          # depois de confirmar a extração contra o PDF
confidence-score: <0.0–1.0>   # honesto: quão diretamente o texto reflete o PDF vs. inferência tua
supersedes: []            # ou [caminho(s)] se substituir página(s) existente(s)
last-verified: <data de hoje, ou omitir se ainda for draft>
```

Corpo: secções mínimas "Propósito"/definição, "Estrutura do documento" (ou síntese, para
conceitos), e sempre uma secção "Fonte" com link de volta ao PDF (documentos) ou às páginas
`documents/*.qmd` citadas (conceitos). Sintetiza — não copies o texto integral do PDF.

## Passo 4 — Atualizar índices e re-renderizar

- Os `index.qmd` de `documents/`/`concepts/` usam `listing: contents: "*.qmd"` — não precisam de
  edição manual para apanhar a página nova, mas confirma que os `fields:` da listagem incluem
  `status`/`confidence-score`/`last-verified` (são só visíveis lá, não na página individual — ver
  `query.md`).
- Corre `quarto render .` na pasta da wiki ativa e confirma que renderiza sem erros e que
  `llms.txt`/`.llms.md` foram gerados para a(s) página(s) nova(s).
- Sugere correr `/lint` depois de ingerir, para apanhar front matter inválido, `supersedes`
  partidos, ou páginas `stale` órfãs introduzidas por este ingest.

PDF a ingerir: $ARGUMENTS
