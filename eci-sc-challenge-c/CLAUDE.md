# eci-sc-challenge-c — LLM Wiki (Quarto) para o domínio SEC (Smart Energy Code)

Knowledge base para o domínio de energia (SEC/SMKI/DCC), seguindo o padrão `artifacts →
ingest → wiki → query/lint` (Karpathy). Fonte imutável: PDFs em `pdfs/`. Destino: páginas
`.qmd` (Quarto, modo Markdown-only) organizadas segundo a estrutura decidida abaixo.

## Decisão de estrutura: Estrutura A — por tipo de conteúdo, granularidade híbrida

```
wiki/
  documents/            # 1 página por PDF de origem (metadata + resumo + link ao artifact)
  concepts/             # 1 página por conceito/entidade, síntese cross-documento
  glossary.qmd          # termos e siglas (DUIS, SMKI, SEC, GBCS, ...)
  index.qmd, _quarto.yml
```

**Nota de estado:** `wiki/` é a versão definitiva (promovida de `_proto/structure-a/` em
2026-07-27, decisão ratificada — ver `docs/quarto-structure-prototype-results.md` Secção 8.7
para o histórico completo e os dados que a sustentam). `/query`, `/lint` e `/ingest` devem
tratar `wiki/` como a wiki corrente. `_proto/` fica só como evidência do protótipo (as 3
estruturas candidatas + os fixtures sintéticos dos testes A/B da Secção 8) — não faz parte da
wiki e não deve ser lido pelo `/query` real.

**Porquê A, e não C:** com um subagente genérico (sem `/query` real), a Estrutura C era mais
barata em descoberta a frio (Secção 5/6.1 do documento acima). Depois de escrever o `/query` real
com a regra de "confiar em `concepts/*.qmd` sem reverificar" (Secção 8.2/8.3), A caiu de 5,6 para
3,04 ficheiros lidos por pergunta ampla — a mais barata das três, sem perder cobertura (Secção
8.4). **A vantagem de custo de A depende inteiramente desta instrução estar implementada no
`/query` — sem ela, a decisão reverte para C.** Nunca apagar essa regra do `/query` sem repetir o
teste.

**Regra de crescimento:** uma página `concepts/*.qmd` não deve acumular facetas ilimitadas — se
passar a agregar >4 fontes distintas, subdividir em sub-secções com âncoras ou em páginas mais
finas (achado da Secção 4.2 do documento de protótipos).

## Schema de front matter

### Campos base (todas as páginas de conteúdo — `type: document`, `type: concept`, `type: glossary`)

| Campo | Obrigatório | Formato | Notas |
|---|---|---|---|
| `title` | sim | string | |
| `type` | sim | `document` \| `concept` \| `glossary` | páginas de listing (`index.qmd` com bloco `listing:`) não usam este schema |
| `categories` | sim | lista | tags livres (ex.: `[security, pki, interface]`) |
| `related` | não | lista de caminhos `.qmd` | cross-links para outras páginas da wiki |
| `last-ingested` | sim | `YYYY-MM-DD` | data em que a página foi gerada/atualizada a partir da fonte |
| `source-pdf` | só `type: document` | caminho relativo | link ao artifact original em `pdfs/` |
| `source-version` | só `type: document` | string | versão/edição do PDF de origem |

### Schema v2 — verificação e proveniência (validado em `docs/quarto-structure-prototype-results.md` Secção 8)

| Campo | Valores | Notas |
|---|---|---|
| `status` | `draft` \| `verified` \| `stale` \| `deprecated` | `stale`/`deprecated` tornam obrigatório um `supersedes` a apontar para esta página noutra página |
| `confidence-score` | `0.0`–`1.0` | confiança da extração/síntese face à fonte; `< 0.7` tem de ser verbalizado pelo `/query` |
| `supersedes` | lista de caminhos `.qmd`, ou `[]` | autoritativo — a página que lista `supersedes: [Y]` prevalece sobre Y em factos sobrepostos |
| `last-verified` | `YYYY-MM-DD` | última verificação humana/agente contra a fonte — distinto de `last-ingested`; não pode ser anterior a `last-ingested` |

**Efeito medido de cada peça (não confundir uma com a outra):**
- O **schema v2** melhora correção/honestidade perante conflito ou baixa confiança (uso confiante
  de facto desatualizado cai de 89% para 16%; conflito não resolvido passa a ser assinalado 80%
  das vezes em vez de 29%) — **não reduz custo** de descoberta.
- A **instrução do `/query`** (confiar em `concepts/*.qmd` sem reverificar) é o que reduz
  ficheiros/tokens lidos — funciona mesmo sem o schema v2 presente.
- Usar os dois juntos é a configuração recomendada: schema para segurança/correção, `/query` para
  custo. Se uma página usa qualquer um dos 4 campos v2, deve usar os 4 (meias-medidas reduzem a
  utilidade do `/query` e do `/lint`).

## Convenções de nomenclatura e linking

- Ficheiros `.qmd` em `kebab-case`, nome descritivo do documento/conceito (ex.:
  `smki-interface-design.qmd`, não `doc3.qmd`).
- Links cross-página relativos ao ficheiro `.qmd` (`[texto](../concepts/smki.qmd)`), nunca URLs
  absolutos nem `@ref`.
- Toda a página `type: document` tem de ter uma secção "Fonte" com link de volta ao PDF original.
- Nunca duplicar nem alterar os PDFs originais em `pdfs/` — são a fonte imutável.

## Ferramenta PDF → Markdown

`markitdown[pdf]` (ver `requirements.txt`) — extração de texto bruta via `pdfminer`, sem deteção
de headings/estrutura. Serve de matéria-prima para escrever as páginas `.qmd` à mão (ou por
agente); não gera hierarquia automaticamente. Alternativas (Docling, Marker, MinerU) não foram
testadas — ver README, Secção "Referências".

## Comandos

- **`/ingest`** (`.claude/commands/ingest.md`) — ingere um PDF novo, gera/atualiza páginas `.qmd`
  seguindo o schema acima.
- **`/query`** (`.claude/commands/query.md`) — responde a perguntas usando só a wiki; respeita
  `supersedes`/`status`/`confidence-score`; confia em `concepts/*.qmd` sem reverificar salvo
  exceções; admite conflito não resolvido em vez de escolher às cegas. Quando a resposta exige
  cruzar mais do que uma página, **cristaliza automaticamente** a síntese numa página
  `concepts/*.qmd` nova ou existente (`query.md`, Passo 5) — sem pedir confirmação, mas
  sinalizando sempre na resposta que a informação foi guardada e porquê. Uma página
  `concepts/*.qmd` pode assim nascer de um `/ingest` ou de uma cristalização do `/query`; em
  qualquer dos casos aplica-se a mesma "Regra de crescimento" abaixo, e a verificação de
  duplicação descrita em `query.md` é a única salvaguarda contra páginas redundantes (não há
  revisão humana antes da escrita).
- **`/lint`** (`.claude/commands/lint.md`) — valida front matter, enums, ranges, `supersedes`
  partidos, páginas `stale` órfãs e deriva semântica entre páginas.

## Histórico e evidência

Todas as decisões acima (estrutura, schema v2, regras do `/query`) foram validadas
estatisticamente (n=46/braço, testes de proporção/Welch t) em
`docs/quarto-structure-prototype-results.md`. Antes de reverter qualquer uma destas decisões,
ler a Secção 8 desse documento — várias conclusões (ex.: "A é mais barato") só são verdadeiras
com as regras do `/query` implementadas; sem elas, a Secção 6.1 mostra que C vencia.
