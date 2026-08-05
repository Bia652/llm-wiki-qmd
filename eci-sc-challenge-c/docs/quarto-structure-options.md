# Opções de Estrutura Quarto — para discussão em equipa

Documento de apoio à decisão da **Secção 0** de [tasks.md](../tasks.md). Lista as alternativas
possíveis para cada eixo de decisão, com prós/contras, para escolher em equipa antes de ingerir
o corpus todo.

Corpus atual (`pdfs/`): 5 especificações do UK Smart Energy Code (SEC) — DCC User Interface,
SMKI Interface Design, SMKI Code of Connection, SMKI Repository Code of Connection, GB Companion
Specification. Documentos de referência técnica/legal, muito inter-referenciados, com muitas
siglas e termos definidos.

---

## Eixo 1 — Tipo de projeto Quarto

| Opção | Descrição | Prós | Contras |
|---|---|---|---|
| **A. `website`** | Site de páginas soltas, navegação livre (sidebar/navbar), `listing:` pages | Navegação não-linear, ideal para consulta por tópico; `listing:` auto-gera índices; mais próximo do conceito "wiki" | Sem noção nativa de "capítulo"/ordem sequencial |
| **B. `book`** | Projeto Quarto tipo livro, capítulos numerados, TOC global, output multi-formato (HTML/PDF/EPUB) | TOC automático e consistente; bom para leitura ponta-a-ponta; export PDF "bonito" | Força ordem linear; menos natural para "entrar direto no conceito X"; renumerar capítulos é chato ao adicionar PDFs |
| **C. Híbrido** | `website` para a wiki de conceitos/documentos + um "book" gerado à parte só para exportar um PDF de referência, se necessário | Flexibilidade total | Mais complexidade de manutenção (2 configs) |

**Recomendação:** A (`website`) — o uso principal é consulta/pergunta, não leitura sequencial.

---

## Eixo 2 — Granularidade das páginas

| Opção | Estrutura | Prós | Contras |
|---|---|---|---|
| **A. 1 página por PDF** | `documents/dcc-user-interface.qmd`, etc. (5 páginas no total) | Simples; mapeamento 1:1 com artifact; fácil rastreabilidade e `lint` | Páginas ficam longas; conceitos partilhados (ex. "SMKI") ficam duplicados em várias páginas |
| **B. 1 página por conceito/entidade** | `concepts/smki.qmd`, `concepts/dcc.qmd`, `concepts/certificate-lifecycle.qmd`... | Conhecimento "compõe" de verdade — cada conceito agrega info de vários PDFs; ótimo para `query` e para o agente | Perde-se a visão "o que diz este PDF, do início ao fim"; mais trabalho de análise no `ingest` |
| **C. Híbrido: documentos + conceitos** | `documents/*.qmd` (overview + secções por PDF) **e** `concepts/*.qmd` (síntese cross-documento) + `glossary.qmd` | Junta rastreabilidade (A) com síntese (B); glossário resolve as siglas repetidas | Mais páginas para manter; precisa de disciplina de linking entre as duas camadas |
| **D. 1 página por secção do PDF** | `documents/dcc-user-interface/01-scope.qmd`, `02-interfaces.qmd`, ... | Granularidade máxima, bom para specs muito longas | Explosão de ficheiros; navegação fica pesada; overhead de manutenção alto para só 5 PDFs |

**Recomendação:** C (híbrido) — melhor equilíbrio para este corpus e volume de documentos (5).

---

## Eixo 3 — Organização de pastas

### Opção 1 — Por tipo de conteúdo (recomendada com granularidade C)
```
wiki/
  documents/
  concepts/
  glossary.qmd
  index.qmd
```

### Opção 2 — Por documento de origem
```
wiki/
  dcc-user-interface/
    overview.qmd
    concepts.qmd
  smki-interface-design/
    overview.qmd
    concepts.qmd
  ...
```
Prós: tudo sobre um PDF fica junto. Contras: conceitos partilhados (SMKI aparece em 4 dos 5 PDFs) ficam espalhados por várias pastas — dificulta síntese cross-documento.

### Opção 3 — Por domínio/tema funcional
```
wiki/
  security/        # SMKI, certificados, chaves
  connectivity/     # DCC, interfaces, mensagens
  compliance/       # Code of Connection, Companion Spec
  glossary.qmd
```
Prós: navegação mais "de negócio", boa para humanos que pensam em termos de área. Contras: fronteiras entre temas nem sempre claras (ex. "Code of Connection" toca segurança e compliance); mais difícil de automatizar no `ingest` (exige classificação manual/IA por tema).

**Recomendação:** Opção 1 — mais previsível para automatizar no `ingest`, e o cross-linking em `concepts/` já resolve a necessidade da Opção 3.

---

## Eixo 4 — Navegação (`_quarto.yml`)

| Opção | Descrição | Prós | Contras |
|---|---|---|---|
| **A. Sidebar única com secções** | `Documents / Concepts / Glossary` numa sidebar só | Simples, um único ponto de navegação | Pode ficar longa com muitos PDFs no futuro |
| **B. Navbar por tópico + sidebar contextual** | Navbar top-level (Documents, Concepts, Glossary), cada um com a sua sidebar | Escala melhor, mais "site profissional" | Mais configuração inicial |
| **C. Listing pages dinâmicas** | `index.qmd` com `listing:` que gera automaticamente cards/tabela de todas as páginas (filtra por `categories`) | Zero manutenção manual do índice à medida que se ingerem PDFs | Exige front matter consistente (categories) em todas as páginas — depende do `lint` |

**Recomendação:** A + C combinadas — sidebar simples (A) e usar `listing:` (C) na página inicial de cada secção para não ter de manter índices à mão.

---

## Eixo 5 — Schema de front matter

| Opção | Campos | Notas |
|---|---|---|
| **A. Mínimo** | `title`, `type` | Rápido de gerar, mas insuficiente para `lint` verificar rastreabilidade |
| **B. Completo (recomendado)** | `title`, `type` (`document`\|`concept`\|`glossary`), `source-pdf`, `source-version`, `categories`, `related`, `last-ingested` | Permite `lint` validar rastreabilidade e links; permite `listing:` filtrar por `categories` |
| **C. Completo + versionamento de conteúdo** | Tudo de B + `status` (`draft`\|`reviewed`) + `reviewed-by` | Útil se houver revisão humana antes de publicar, mas acrescenta processo manual |

**Recomendação:** B — suficiente para automação (`ingest`/`lint`/`listing`) sem processo manual extra.

---

## Eixo 6 — Convenção de linking cruzado

| Opção | Descrição | Prós | Contras |
|---|---|---|---|
| **A. Links relativos Markdown** `[SMKI](../concepts/smki.qmd)` | Nativo, simples, funciona em qualquer editor Markdown | Quebra silenciosamente se se renomear/mover ficheiros (só o `lint` apanha) |
| **B. Front matter `related:`** + geração automática de secção "Ver também" | Explícito, fácil de validar por `lint` (basta confirmar que o ficheiro existe) | Não cobre links inline dentro do texto |
| **C. Combinação A + B** | Links inline onde faz sentido no texto + `related:` no front matter para links estruturados "Ver também" | Melhor cobertura, mais robusto para `lint` | Duas convenções para manter disciplina |

**Recomendação:** C — inline para contexto, `related:` para navegação estruturada e para o `lint` verificar facilmente.

---

## Eixo 7 — Geração `llms.txt` / consumo por IA

| Opção | Descrição | Prós | Contras |
|---|---|---|---|
| **A. `llms-txt: true` simples** | Usa a feature nativa do Quarto tal-e-qual, sem afinação | Zero esforço | Pode incluir todas as páginas sem hierarquia/priorização |
| **B. `llms-txt` com curadoria** | Configurar quais páginas entram no índice curto vs `llms-full.txt`, priorizando `concepts/` sobre `documents/` | Índice mais denso em síntese, menos ruído para o agente | Precisa de configuração/manutenção manual |

**Recomendação:** B, mas começar com A e refinar depois de ver o output real.

---

## Resumo da recomendação (para ratificar em equipa)

1. **Tipo:** `website`
2. **Granularidade:** híbrida — `documents/` + `concepts/` + `glossary.qmd`
3. **Pastas:** por tipo de conteúdo (Eixo 3, Opção 1)
4. **Navegação:** sidebar simples + `listing:` automático
5. **Front matter:** schema completo (Eixo 5, Opção B)
6. **Linking:** inline + `related:` no front matter
7. **`llms.txt`:** ativar já, curar depois de ver o resultado com o corpus real

## Ação seguinte
Depois de a equipa validar/ajustar esta escolha, registar a decisão final em [CLAUDE.md](../CLAUDE.md)
e marcar a Secção 0 de [tasks.md](../tasks.md) como concluída.
