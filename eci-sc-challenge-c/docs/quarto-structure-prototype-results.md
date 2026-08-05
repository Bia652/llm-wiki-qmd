# Resultados do protótipo — 3 estruturas Quarto testadas

Continuação de [quarto-structure-options.md](quarto-structure-options.md). Em vez de decidir só
"no papel", construímos os 3 candidatos como projetos Quarto reais (em `_proto/`, área de
protótipo — não faz parte da wiki final), com conteúdo extraído a sério de PDFs reais do
corpus, e testámo-los em profundidade: critérios qualitativos, eficiência de custo (tokens/
tempo, com e sem oráculo), e qualidade das respostas (108+ respostas avaliadas).

## Sumário executivo

**Decisão sob 2 critérios explícitos, por esta ordem: 1º coverage/correção, 2º custo de tokens/
tempo em produção real → Estrutura C.**

| Critério | Resultado |
|---|---|
| 1. Coverage/correção (108 respostas avaliadas) | **Empate total** — 100% cobertura, 0 erros nas 3 estruturas |
| 2. Custo tokens/tempo, cenário de produção real (descoberta a frio, n=46, sem oráculo) | **C vence**: 34 206 tokens (vs. 35 297 de A, 35 254 de B, p ≤ 0,0001) e 26 622 ms (mais rápida que B, p=0,004; vs. A na fronteira, p=0,047) |

O critério 1 empata, por isso o critério 2 decide, e favorece **C**. Isto contraria a intuição
inicial (e os testes preliminares "com oráculo", onde A parecia claramente mais barata) — a
razão da inversão está detalhada na Secção 5, e é o resultado mais importante deste documento.

**Nota de proveniência:** os testes "com oráculo" (Secção 3) foram uma escolha metodológica da
IA, tomada sem ser pedida, ao desenhar os primeiros testes estatísticos — e, sozinhos, davam uma
conclusão diferente (A mais barata). Só ao correr manualmente o teste sem oráculo (Secção 5), a
pedido explícito do utilizador, é que se confirmou que essa conclusão inicial não generalizava
para o uso real. Ver nota completa no início da Secção 3.

**Fatores fora desta decisão de 2 critérios** (documentados, mas não pedidos como critério): A
evita duplicação/deriva semântica entre páginas (B e C não evitam) e exige menos manutenção ao
ingerir do que B. Se a equipa quiser pesar estes fatores como 3º critério, a decisão pode reverter
para A — ver Secção 6.

---

## 1. Metodologia

### 1.1 Setup

- **Tooling instalado:** Quarto CLI 1.9.38 (zip portátil, sem alterar o PATH do sistema —
  binário em `%LOCALAPPDATA%\Programs\quarto\bin`) e `markitdown[pdf]` (ver `requirements.txt`).
- **Conversão PDF → Markdown:** `python -m markitdown <pdf> -o _proto/raw-md/<nome>.md`.
  Nota lateral (relevante para a Secção 3 do `tasks.md`): o `markitdown` faz extração de texto
  bruta (via `pdfminer`), sem deteção de headings/estrutura — dá um `.md` "achatado" que serviu
  de matéria-prima para escrever as páginas `.qmd` à mão, mas não gera hierarquia automaticamente.
  Isto é um input para a escolha de ferramenta na Secção 3 do `tasks.md`, não uma conclusão
  definitiva (não testámos Docling/Marker/MinerU).
- **Corpus do protótipo:** começámos com 1 PDF (`SEC-Appendix-AD-DCC-User-Interface-Specification-v5.4`),
  depois um **2º PDF** (`SEC-Appendix-M-SMKI-Interface-Design-Specification-v8.0` — os dois
  partilham um conceito central, **SMKI**, o pior caso realista para testar duplicação), e por
  fim o **corpus completo de 5 PDFs** (mais SMKI Code of Connection, SMKI Repository Code of
  Connection, GB Companion Specification).
- **3 candidatos construídos** (todos com o mesmo Eixo 4 — sidebar `docked` + `listing:` +
  sidebar `auto:` por pasta — e o mesmo Eixo 5/6/7, para isolar apenas a variável Eixo 2/3):
  - **A — por tipo de conteúdo** (`documents/` + `concepts/` + `glossary.qmd`, granularidade híbrida)
  - **B — por documento de origem** (pasta por PDF com `overview.qmd` + `concepts.qmd`)
  - **C — baseline, 1 página por PDF** (sem camada de conceitos, sem glossário separado)

Todos renderizaram sem erros (`quarto render .`) e geraram `llms.txt`/`.llms.md` (a feature
`llms-txt: true` nativa do Quarto; não gerou `llms-full.txt` nestes protótipos — parece
depender de configuração adicional que não explorámos).

### 1.2 Limitação importante: `ingest`/`lint`/`query` não são reais neste protótipo

Os comandos `.claude/commands/ingest.md`, `lint.md` e `query.md` (Secção 1 do `tasks.md`) **estão
vazios** — ainda não foram escritos. Todo o "ingest" e "query" usados neste protótipo e nos
testes estatísticos foram **simulados manualmente**, não invocações de comandos reais:

- **"Ingest":** as páginas `.qmd` foram escritas à mão a partir do `.md` bruto do `markitdown`,
  não geradas por um comando `/ingest`.
- **"Query" nos testes "com oráculo"** (Secção 3): em cada corrida, disse-se explicitamente ao
  subagente **quais ficheiros ler** — não houve descoberta/pesquisa. Isto mede o custo de *ler*
  a resposta certa depois de já se saber onde ela está, não o custo de a *encontrar*.
- **"Query" no teste de descoberta a frio** (Secção 5): aqui sim, sem oráculo — o subagente só
  recebeu a pergunta e o índice `llms.txt`, tendo de decidir sozinho o que ler.
- **"Lint":** nunca foi corrido — não se validou front matter, links partidos nem consistência
  entre páginas por nenhum mecanismo automático.

O `.claude/commands/query.md` real, tal como descrito no README/`tasks.md`, é um comando leve no
estilo Karpathy — sem embeddings, vector search ou índice pré-construído. **Isto significa que o
cenário de descoberta a frio (Secção 5) é o que prevê o comportamento real em produção**, não os
testes com oráculo (Secção 3) — importante ter isto em mente ao ler o resto do documento.

---

## 2. Critérios qualitativos de estrutura

### 2.1 Critério 1 — Facilidade de manutenção / ingestão incremental

Medido a sério: adicionámos o 2º PDF a cada estrutura e contámos os ficheiros tocados.

| Estrutura | Ficheiros novos | Ficheiros editados | Config editada? (`_quarto.yml`/`index.qmd`) | Duplicação de conteúdo introduzida |
|---|---|---|---|---|
| **A** | 1 (`documents/smki-interface-design.qmd`) | 1 (`concepts/smki.qmd`, para acrescentar a síntese) | **Não** (sidebar `auto:` + `listing:` apanham a página nova sozinhos) | Nenhuma — o conceito SMKI continua numa única página canónica |
| **B** | 2 (`smki-interface-design/overview.qmd` + `concepts.qmd`) | 2 (`_quarto.yml` — nova secção na sidebar; `index.qmd` — nova entrada no `listing:`) | **Sim, 2 ficheiros** | Sim — duas páginas `concepts.qmd` diferentes (uma por pasta) definem "SMKI" de forma distinta e não-sincronizada |
| **C** | 1 (`documents/smki-interface-design.qmd`) | 0 | **Não** | Sim — a definição de SMKI e 4 entradas de glossário ficam repetidas (com texto ligeiramente diferente) entre as duas páginas |

**Achado principal:** o `sidebar: contents: auto: "<pasta>"` do Quarto (não estava no plano
original, descoberto ao testar) elimina a manutenção manual de índice/sidebar **desde que a
pasta já exista**. A diferença real entre A e C não é o nº de ficheiros tocados (ambos = 1 no
caso simples), é que **C acumula duplicação de conteúdo partilhado a cada novo PDF que toca num
conceito já coberto**, enquanto **A tem sempre um único sítio para atualizar esse conceito**. B é
estritamente pior nos dois eixos: mais ficheiros tocados (exige sempre uma nova secção manual na
sidebar e no `listing:` raiz porque cada PDF é uma pasta nova) **e** sofre da mesma duplicação/
fragmentação que C.

### 2.2 Critério 2 — Qualidade da navegação humana no site renderizado

Renderizámos os 3 e navegámos no browser (via servidor local, screenshots tirados).

- **A:** sidebar limpa com secções "Documents" / "Concepts" / "Glossary". A página
  `concepts/smki.qmd` mostra claramente as duas fontes ("Uso na DUIS" + "Emissão de
  certificados") na mesma página, com TOC lateral e breadcrumb. Um humano que pesquise "SMKI"
  encontra uma resposta só, completa. Defeito menor: o breadcrumb duplica o nome da secção
  (`Concepts > Concepts > SMKI...`), resolúvel ajustando os títulos de `contents: auto`.
- **B:** a listing raiz mostra `Conceitos — DCC User Interface` e `Conceitos — SMKI Interface
  Design` como duas entradas distintas e sem relação visível entre si — um humano só percebe que
  se sobrepõem se abrir as duas páginas. Os títulos de secção da sidebar derivam do nome da pasta
  (`Dcc User Interface`, `Smki Interface Design`), com capitalização estranha.
- **C:** navegação mais pobre à medida que o corpus cresce — não há forma de "entrar direto no
  conceito X" sem abrir um documento inteiro; a listing de `documents/` acumula `categories`
  cada vez mais sobrepostas sem um sítio central para elas.

**Vencedor:** A.

### 2.3 Critério 3 — Qualidade do `llms.txt` / `.llms.md` gerado

`llms.txt` do Quarto é **sempre uma lista plana** nas 3 estruturas (não agrupa por secção/tipo,
confirmando a preocupação do Eixo 7 Opção A). Isso não varia entre A/B/C — a curadoria (Eixo 7
Opção B) continua a ser necessária independentemente da estrutura de pastas.

Onde a estrutura *importa* é na qualidade de cada `.llms.md` individual:

- **A:** `concepts/smki.llms.md` é uma página de síntese autónoma e não-redundante.
- **B:** um agente que leia só `dcc-user-interface/concepts.llms.md` fica com uma definição
  incompleta (falta a emissão de certificados) — teria de saber que existe também
  `smki-interface-design/concepts.llms.md` e cruzar as duas.
- **C:** cada `documents/*.llms.md` duplica a definição de SMKI com **redação ligeiramente
  diferente** em cada ficheiro — risco de **deriva semântica**, não só duplicação.

**Vencedor:** A.

---

## 3. Eficiência de custo — cenário com oráculo

> **Nota de proveniência metodológica.** A decisão de testar "com oráculo" (dizer ao subagente
> exatamente que ficheiro ler, em vez de o deixar descobrir sozinho) foi uma escolha **minha**
> (da IA), tomada sem ser explicitamente pedida, ao desenhar os primeiros testes estatísticos de
> tokens/tempo. Fazia sentido como forma de isolar "quanto custa ler X" da questão "quanto custa
> encontrar X" — mas, sem o testar de outra forma, ficou implícito que este era o cenário
> representativo de uso real, quando não é. **Só ao correr manualmente o teste sem oráculo
> (Secção 5), a pedido explícito do utilizador, é que se percebeu que a conclusão baseada apenas
> nestes testes com oráculo era enganadora**: aqui A parece claramente mais barata (Secção 3.2–
> 3.5), mas em descoberta real (Secção 5) é C que sai mais barata — o oposto. Os números desta
> secção mantêm-se corretos e úteis (isolam o custo de leitura), mas **não devem ser lidos como
> previsão do comportamento do `/query` real** sem a correção da Secção 5.

### 3.1 Heurística determinística (chars/4)

Para tornar o Critério 3 mensurável, medimos quantos tokens (heurística `chars/4`, não um
tokenizer real) um agente precisa de **ler** para responder por completo a "O que é o SMKI e
como é usado?" — o conjunto mínimo de ficheiros `.qmd` que cobre a resposta completa.

| Estrutura | Ficheiros a ler p/ resposta completa | Tokens da resposta | Tokens do corpus total | % do corpus lido |
|---|---|---|---|---|
| **A** | 1 (`concepts/smki.qmd`) | ~430 | ~1463 | 29% |
| **B** | 2 (`dcc-user-interface/concepts.qmd` + `smki-interface-design/concepts.qmd`) | ~558 | ~1201 | 46% |
| **C** | 2 (`documents/dcc-user-interface.qmd` + `documents/smki-interface-design.qmd` inteiros) | ~1125 | ~1125 | 100% |

A Estrutura A, com apenas 2 PDFs, já é 30% mais barata que B e mais de 2,5× mais barata que C —
mas o ponto mais importante é como isto **escala** com o corpus completo (vários PDFs a tocar
SMKI). Se um conceito for tocado por `k` documentos:

- **A:** custo fica **constante** (~430 tokens) — o agente lê só `concepts/smki.qmd`, seja qual
  for o `k`.
- **B:** custo cresce **linearmente com `k`** (~279 tokens por pasta de documento tocada), cada
  fragmento parcial/redundante com os outros.
- **C:** custo cresce **linearmente com `k`**, com um fator maior por documento (~562 tokens —
  a página inteira, não só o trecho relevante).

Com `k=3` (ilustração): A ≈ 430 tokens (constante), B ≈ 837, C ≈ 1687 — quase 4× mais caro que A.

### 3.2 Medição real de tokens, corpus 2 PDFs (k=2), n=46

Tokens reais (`subagent_tokens`, reportado pela infraestrutura de agentes, não estimado),
n=46 corridas independentes por estrutura (138 subagentes), cada uma lendo só o conjunto mínimo
de ficheiros e respondendo à mesma pergunta:

| Estrutura | n | Média | Mediana | Desvio-padrão |
|---|---|---|---|---|
| **A** | 46 | 29 228 | 29 230 | 33 |
| **B** | 46 | 29 874 | 29 879 | 37 |
| **C** | 46 | 31 113 | 31 113 | 51 |

Mann-Whitney U, Welch's t-test, Cohen's *d*:

| Comparação | Mann-Whitney U (p) | Welch t (df) | Cohen's d |
|---|---|---|---|
| A vs B | U = 0 → **p < 0,0001** | t = -88,8 (df=88,7) | -18,5 (enorme) |
| A vs C | U = 0 → **p < 0,0001** | t = -209,8 (df=76,4) | -43,8 (enorme) |
| B vs C | U = 0 → **p < 0,0001** | t = -132,7 (df=81,7) | -27,7 (enorme) |

Separação total: nenhum dos 46 valores de A se sobrepõe a nenhum de B ou C (`U = 0`). Desvio-
padrão minúsculo (33–51) face à diferença entre grupos (~650–1900) — efeito enorme e estável:
**A < B < C consistentemente**, com este corpus e esta pergunta.

### 3.3 Validação da escala: corpus completo (5 PDFs, k=4), n=46

Ingerimos os 3 PDFs em falta (SMKI Code of Connection, SMKI Repository Code of Connection, GB
Companion Specification) e repetimos a medição. Das 5 PDFs, **4 tocam em SMKI**; a GB Companion
não menciona SMKI (0 ocorrências, confirmado por grep) e fica corretamente fora do conjunto
mínimo de resposta.

| Estrutura | Ficheiros a ler (k=4) | n | Média tokens | Desvio-padrão |
|---|---|---|---|---|
| **A** | 1 (`concepts/smki.qmd`, agora sintetizando 4 fontes) | 46 | 30 929 | 42 |
| **B** | 4 (`concepts.qmd` de cada um dos 4 documentos) | 46 | 32 256 | 36 |
| **C** | 4 (páginas de documento inteiras) | 46 | 34 122 | 37 |

| Comparação | Mann-Whitney U (p) | Welch t (df) | Cohen's d |
|---|---|---|---|
| A vs B | U = 0 → **p < 0,0001** | t = -164,0 (df=88,0) | -34,2 (enorme) |
| A vs C | U = 0 → **p < 0,0001** | t = -387,1 (df=89,0) | -80,7 (enorme) |
| B vs C | U = 0 → **p < 0,0001** | t = -244,7 (df=89,8) | -51,0 (enorme) |

**A hipótese de escala confirma-se empiricamente.** Deltas entre estruturas, corpus 2 PDFs
(k=2) vs 5 PDFs (k=4):

| Delta (tokens) | k=2 | k=4 | Cresceu ~2×? |
|---|---|---|---|
| B − A | 646 | 1 328 | Sim (×2,06) |
| C − A | 1 885 | 3 194 | Sim (×1,69) |
| C − B | 1 239 | 1 866 | Sim (×1,51) |

O custo de A cresceu pouco (+1 701 tokens, essencialmente o conteúdo extra sintetizado na mesma
página única) enquanto o "prémio" de B e C sobre A mais do que duplicou ao passar de k=2 para
k=4 — exatamente o comportamento previsto: A fica plano por conceito, B e C escalam
linearmente com `k`.

### 3.4 Medição real de tempo, corpus 2 PDFs (k=2), n=46

Subagente isolado por estrutura, auto-cronometrado com `date` (Bash) antes/depois de ler os
ficheiros, corrido via `Workflow` em paralelo (50 por estrutura inicialmente).

**Bug de instrumentação encontrado e corrigido antes de analisar:** 9 de 150 corridas (6%)
devolveram valores inválidos (timestamps de época em vez de durações, ou valores impossivelmente
baixos), causados por o estado da shell não persistir entre chamadas Bash separadas
(`START=$(date...)` numa chamada, `END=$(date...)` noutra — a variável já não existia). Excluídos
(critério: 1000 ms < valor ≤ 1 000 000 ms), truncando todos para **n=46** válidos por estrutura.

| Estrutura | n | Média | Mediana | Desvio-padrão |
|---|---|---|---|---|
| **A** | 46 | 8 401 ms | 7 804 ms | 4 304 ms |
| **B** | 46 | 8 455 ms | 8 341 ms | 4 128 ms |
| **C** | 46 | 8 771 ms | 8 403 ms | 4 681 ms |

| Comparação | Mann-Whitney U (p) | Welch t (df) | Cohen's d |
|---|---|---|---|
| A vs B | p = 0,490 (não significativo) | t = -0,06 (df=89,8) | -0,01 (negligenciável) |
| A vs C | p = 0,569 (não significativo) | t = -0,39 (df=89,4) | -0,08 (negligenciável) |
| B vs C | p = 0,827 (não significativo) | t = -0,34 (df=88,6) | -0,07 (negligenciável) |

**Leitura honesta:** com amostra balanceada (n=46), **não há diferença de tempo estatisticamente
detetável entre A, B e C** com este corpus pequeno — a latência é dominada por overhead fixo do
agente que não escala de forma detetável com ler 1 ou 2 ficheiros pequenos.

### 3.5 Medição real de tempo, corpus completo (5 PDFs, k=4), n=46

Mesma medição, agora A lê 1 ficheiro e B/C leem os 4 que tocam SMKI.

| Estrutura | n | Média | Mediana | Desvio-padrão |
|---|---|---|---|---|
| **A** | 46 | 8 543 ms | 7 921 ms | 2 584 ms |
| **B** | 46 | 10 644 ms | 10 556 ms | 1 478 ms |
| **C** | 46 | 11 020 ms | 10 621 ms | 2 587 ms |

| Comparação | Mann-Whitney U (p) | Welch t (df) | Cohen's d |
|---|---|---|---|
| A vs B | **p = 1,7 × 10⁻⁷** | t = -4,79 (df=71,6) | -1,00 (grande) |
| A vs C | **p = 2,6 × 10⁻⁶** | t = -4,59 (df=90,0) | -0,96 (grande) |
| B vs C | p = 0,809 (não significativo) | t = -0,85 (df=71,6) | -0,18 (pequeno) |

**A diferença de tempo, que não era detetável com k=2, torna-se estatisticamente significativa
com k=4** (A mais rápida que B e C). Ler 1 ficheiro em vez de 2 não pesa o suficiente face ao
overhead fixo do agente, mas ler 1 em vez de 4 já pesa. B e C não diferem entre si em tempo
(p=0,81) — ambas leem 4 ficheiros, só a *dimensão* de cada um difere.

**Síntese da Secção 3 (só válida no cenário com oráculo — ver Secção 5 para o cenário real):**
nos testes com oráculo, A é significativamente mais barata em tokens e (com corpus suficiente)
mais rápida do que B e C, e essa vantagem cresce com o tamanho do corpus.

---

## 4. Qualidade das respostas: cobertura, correção, precisão, resistência a alucinação

Os testes da Secção 3 medem **custo**, não **qualidade**. Esta secção pontua as respostas reais
dadas pelos subagentes contra checklists do que uma resposta completa e correta deveria conter.

### 4.1 Pergunta ampla ("O que é o SMKI e como é usado?"), ambos os corpus

**Checklist, corpus 2 PDFs:** (1) definição correta do SMKI; (2) uso na DUIS — ECDSA P-256 e as
3 chaves privadas nomeadas com propósito; (3) emissão de certificados — os 3 canais de CSR
nomeados; (4) citação correta das 2 fontes.

**Checklist, corpus 5 PDFs** — 2 itens adicionais: (5) acesso aos SMKI Services — estatuto ARO +
credenciais IKI, via DCC Gateway Connection; (6) acesso ao SMKI Repository — via Gateway/Service
Desk/Portal.

| Corpus | A | B | C |
|---|---|---|---|
| 2 PDFs (n=5 por estrutura, itens 1–4) | 5/5 respostas, 4/4 itens (100%), 0 erros | 5/5, 4/4 (100%), 0 erros | 5/5, 4/4 (100%), 0 erros |
| 5 PDFs (n=3 por estrutura, itens 1–6) | 3/3, 6/6 (100%), 0 erros | 3/3, 6/6 (100%), 0 erros | 3/3, 6/6 (100%), 0 erros |

### 4.2 Generalização — 3 perguntas com cenários diferentes (n=3 por estrutura, corpus 5 PDFs)

A pergunta ampla acima é o pior caso possível para B/C (síntese cross-documento). Testámos mais
3 perguntas para verificar se a qualidade e o custo generalizam:

| Pergunta | Cenário | Ficheiros mín. — A | Ficheiros mín. — B | Ficheiros mín. — C |
|---|---|---|---|---|
| **Q2** — "Que tipos de Service Request existem na DUIS?" | Conceito só num documento (sem partilha) | 1 (`concepts/duis-messaging.qmd`, 177 tok) | 1 (`dcc-user-interface/concepts.qmd`, 306 tok) | 1 (`documents/dcc-user-interface.qmd`, 645 tok) |
| **Q3** — "O que é a GBCS e que Dispositivos a cumprem?" | Documento sem sobreposição nenhuma (controlo) | 1 (`documents/gb-companion.qmd`, 247 tok) | 1 (`gb-companion/concepts.qmd`, **97 tok**) | 1 (`documents/gb-companion.qmd`, 180 tok) |
| **Q4** — "Que credenciais dão acesso aos SMKI Services?" | Facto concentrado numa só fonte, dentro de um conceito partilhado | 1 (`concepts/smki.qmd`, **682 tok**) | 1 (`smki-coc/concepts.qmd`, 164 tok) | 1 (`documents/smki-coc.qmd`, 316 tok) |

**Cobertura/correção:** Q2, Q3 e Q4 tiveram 3/3 respostas com cobertura total e 0 erros, nas 3
estruturas — qualidade continua perfeita.

**Custo em tokens — aqui a generalização falha, e é honesto dizê-lo:**

- **Q2 (conceito isolado):** A continua a ganhar (177 vs 306 vs 645) — não por partilha
  cross-documento, mas porque a sua página de conceito é mais focada do que a página
  multi-conceito de B ou a página de documento inteira de C.
- **Q3 (controlo, zero sobreposição):** **B ganha** (97 tokens) — A não tem página `concepts/`
  dedicada para GBCS (nunca precisou, por não partilhar com outros documentos), por isso cai de
  volta para a página de documento inteira, mais pesada do que a página de conceito enxuta de B.
- **Q4 (facto estreito dentro de conceito partilhado):** **B ganha claramente** (164 vs 682 em
  A — A fica ~4× mais caro). A página canónica de A agrega tudo sobre SMKI (4 facetas); uma
  pergunta sobre 1 só faceta obriga a ler — e a filtrar explicitamente nas respostas — conteúdo
  irrelevante que B e C não têm.

**Conclusão desta secção:** a vantagem de tokens de A **não é universal mesmo com oráculo** —
depende de a pergunta exigir síntese cross-documento (o caso mais importante, que mais escala
com o corpus) ou incidir sobre um conceito com página própria fina. Para perguntas muito
estreitas, B pode ser mais barata. Isto aponta para um refinamento: **a página de conceito de A
não deve crescer sem limite** — se acumular demasiadas facetas (como `concepts/smki.qmd`, que
passou a agregar 4 fontes), subdividir em sub-secções ou páginas mais finas.

### 4.3 Perguntas profundas: precisão, resistência a alucinação, honestidade epistémica

As perguntas anteriores testavam sobretudo *recall*. Estas 3 perguntas usam critérios de
avaliação mais exigentes (n=3 por estrutura, corpus 5 PDFs):

| Pergunta | O que testa | Critério de sucesso |
|---|---|---|
| **Q5** — "Uma Organização quer submeter um Device CSR em lote e, separadamente, uma Organisation CSR. Podem ambas ser submetidas pela mesma interface?" | **Precisão** numa distinção subtil de uma tabela | Reconhecer que só o SMKI Portal aceita os dois tipos de CSR; os 2 Web Services dedicados só aceitam Device CSR |
| **Q6** — "Qual é o critério exato que distingue um Device Request 'Critical' de um 'Non Critical'?" | **Resistência a alucinação** — a informação não está nas fontes ingeridas | Reconhecer explicitamente que o critério não é especificado, sem inventar uma resposta plausível |
| **Q7** — "As 3 chaves privadas da DUIS — através de que canal específico da SMKI Interface Design cada uma obtém o certificado?" | **Honestidade epistémica** sobre uma lacuna de inferência cross-documento | Reconhecer que os documentos nunca cruzam as duas tabelas; distinguir especulação de facto declarado |

| Pergunta | A | B | C |
|---|---|---|---|
| Q5 (precisão) | 3/3 corretas | 3/3 corretas | 3/3 corretas |
| Q6 (anti-alucinação) | 3/3 recusaram inventar | 3/3 recusaram inventar | 3/3 recusaram inventar |
| Q7 (honestidade epistémica) | 3/3 distinguiram facto de inferência | 3/3 distinguiram facto de inferência | 3/3 distinguiram facto de inferência |

**Zero falhas em 27 respostas**, nas 3 estruturas: nenhuma inventou o critério Critical/Non-
Critical, nenhuma afirmou com falsa certeza uma correspondência chave→canal não sustentada, e
todas leram a tabela de CSR corretamente (nenhuma confundiu "Batched" com um canal que também
aceita Organisation CSR).

Custo em tokens (heurística), mais uma confirmação do padrão da Secção 4.2:

| Pergunta | A | B | C | Vencedor |
|---|---|---|---|---|
| Q5 (facto estreito, dentro de conceito partilhado) | 682 | **252** | 480 | B |
| Q6 (conceito isolado, 1 documento) | **177** | 306 | 645 | A |
| Q7 (síntese cross-documento) | **682** | 558 | 1125 | A (mas B mais perto desta vez) |

### 4.4 Exemplos ilustrativos — pergunta e resposta real, lado a lado

**Exemplo 1 — síntese ampla cross-documento.** Pergunta: *"O que é o SMKI e como é usado?"*

> **Estrutura A** (leu só `concepts/smki.qmd`): "O SMKI (Smart Meter Key Infrastructure) é
> definido como a infraestrutura de chave pública (PKI) que certifica as chaves usadas para
> assinar digitalmente as comunicações no ecossistema SEC. O documento organiza a informação em
> quatro áreas: **1. Uso na DUIS** — cláusula 3.3, três chaves privadas. **2. Emissão de
> certificados** — SMKI Interface Design Spec, Secção L4, três canais de CSR. **3. Ligação aos
> SMKI Services** — SMKI Code of Connection, Secção L4.5, ARO + credenciais IKI. **4. Acesso ao
> SMKI Repository** — SMKI Repository Code of Connection, Secção L6.5, via Gateway/Service
> Desk/Portal."

> **Estrutura B** (leu 4 ficheiros `concepts.qmd`, um por documento): "...definido de forma
> idêntica em todos os quatro documentos... **Nota do próprio subagente:** 'esta definição já
> existe em `dcc-user-interface/concepts.qmd`... como na Estrutura B não há uma página de
> conceitos partilhada entre pastas, a definição teve de ser duplicada em cada pasta de
> documento.' [segue a mesma síntese em 4 partes]."

> **Estrutura C** (leu 4 páginas de documento inteiras): "...definido de forma idêntica em todos
> os quatro documentos... Note-se que os próprios documentos assinalam, em notas de protótipo,
> que esta definição está duplicada em cada ficheiro — não existe uma camada partilhada de
> conceitos/glossário nesta estrutura... [segue a mesma síntese em 4 partes]."

Conteúdo final equivalente nas 3 (cobertura 100%), mas só B e C precisaram de **auto-assinalar a
duplicação** que leram — em A essa duplicação simplesmente não existe para ser assinalada.

**Exemplo 2 — resistência a alucinação (Q6).** Pergunta: *"Qual é o critério exato que distingue
um Device Request 'Critical' de um 'Non Critical'?"*

> **A:** "O ficheiro não especifica o critério exato... Conclusão: com base exclusivamente neste
> ficheiro, não é possível responder à pergunta sem inventar informação."

> **B:** "O ficheiro não especifica um critério exato... não menciona nada sobre tipo de comando,
> risco associado, necessidade de autorização adicional, ou qualquer outro critério."

> **C:** "O ficheiro **não especifica** o critério exato... não é possível responder à pergunta
> com base apenas neste ficheiro; a informação simplesmente não está presente nele."

As 3 recusaram inventar uma resposta plausível, apesar de ser uma tentação natural para um
modelo de linguagem completar a frase com algo verosímil.

**Exemplo 3 — precisão numa distinção subtil (Q5).** Pergunta: *"Uma Organização quer submeter
um Device CSR em lote e, separadamente, uma Organisation CSR. Podem ambas ser submetidas pela
mesma interface?"*

> **A:** "Sim — ambas podem ser submetidas pela mesma interface, mas apenas se essa interface for
> o SMKI Portal... os outros dois canais são exclusivos a Device CSR... nenhum dos dois aceita
> Organisation CSR."

> **B:** "Sim. Existe uma única interface que aceita ambos os tipos de submissão — o SMKI
> Portal... As outras duas interfaces listadas são de propósito único."

> **C:** "Sim, é possível — mas apenas se ambas forem submetidas através do SMKI Portal... Os
> outros dois canais são especializados por tipo/modo de CSR e não servem o outro caso."

As 3 leram a tabela com a mesma precisão — nenhuma cometeu o erro fácil de assumir que "Batched"
aceitaria também Organisation CSR só porque o SMKI Portal aceita ambos.

### 4.5 Síntese da Secção 4

Juntando todas as perguntas testadas (pergunta ampla ×2 corpus, Q2–Q4, Q5–Q7): **108 respostas
avaliadas, 0 erros factuais, 0 alucinações, 100% de cobertura/precisão, nas 3 estruturas, sem
exceção.** A escolha entre A/B/C não é, em nenhum cenário testado, uma escolha entre "melhor" e
"pior" na qualidade da resposta.

---

## 5. Eficiência em cenário real de produção — teste de descoberta a frio (n=46)

Todos os testes da Secção 3 tinham um "oráculo" (dizíamos ao agente que ficheiro ler). Esta
secção fecha esse gap: o subagente recebe só a pasta `_site/` (o site publicado, com `llms.txt`)
e a pergunta, e decide sozinho, a partir do índice, que páginas `.llms.md` ler — sem ajuda. Como
já estabelecido na Secção 1.2, **este é o cenário que reflete o `/query` real**. Corrido com o
mesmo rigor do resto do documento: n=46 por estrutura (138 corridas).

### Resultados

| Estrutura | Ficheiros lidos (média ± dp) | Tokens (média ± dp) | Duração (média ± dp) |
|---|---|---|---|
| **A** | 5,6 ± 1,3 | 35 297 ± 1 088 | 27 626 ± 5 203 ms |
| **B** | 8,4 ± 0,7 | 35 254 ± 660 | 28 877 ± 4 754 ms |
| **C** | 5,0 ± 1,2 | 34 206 ± 1 410 | 26 622 ± 7 527 ms |

| Comparação | Métrica | Mann-Whitney U (p) | Cohen's d |
|---|---|---|---|
| A vs B | ficheiros lidos | **p < 0,0001** | -2,67 (enorme, B lê mais) |
| A vs C | ficheiros lidos | **p = 0,0006** | 0,50 (médio, A lê mais que C) |
| B vs C | ficheiros lidos | **p < 0,0001** | 3,45 (enorme, B lê muito mais) |
| A vs B | tokens | p = 0,082 (não significativo) | 0,05 (negligenciável) |
| A vs C | tokens | **p = 0,0001** | 0,87 (grande, **C mais barata**) |
| B vs C | tokens | **p < 0,0001** | 0,95 (grande, **C mais barata**) |
| A vs B | tempo | p = 0,241 (não significativo) | -0,25 |
| A vs C | tempo | p = 0,047 (fronteira) | 0,16 (pequeno) |
| B vs C | tempo | **p = 0,004** | 0,36 (médio, **C mais rápida**) |

### A amostra maior inverteu a leitura inicial — o resultado mais honesto deste documento

Um piloto inicial com n=5 sugeria que A continuava a ser a mais rápida em descoberta real, tal
como nos testes com oráculo. **Com n=46, essa leitura não se sustenta.** Os dados que sobrevivem
ao escrutínio estatístico dizem o oposto do que a intuição (e a amostra pequena) sugeriam:

- **Em tokens, C é a mais barata** (34 206, contra 35 297 de A e 35 254 de B — ambas
  estatisticamente distintas de C, p ≤ 0,0001). B e A ficam estatisticamente empatadas entre si
  (p = 0,082).
- **Em tempo, C não é pior** — significativamente mais rápida do que B (p = 0,004); a diferença
  A vs. C está na fronteira da significância (p = 0,047, favorecendo C).
- **A lê mais ficheiros do que C em média** (5,6 vs. 5,0, p = 0,0006) — o oposto do que a
  intuição sugeriria para uma estrutura com página de conceito dedicada.

**Porquê:** em várias corridas de A, o subagente encontrava `concepts/smki.llms.md` (já com a
síntese completa) mas **não confiava nela sozinho** e ia verificar os 4 documentos de origem —
ficheiros/tokens a mais que uma leitura "ideal" não precisaria. Em C, uma página de documento
inteira contém tipicamente contexto suficiente para o subagente se dar por satisfeito com poucas
leituras. B continua a ser a que lê sistematicamente mais ficheiros (8,4 em média), por ter de
abrir as 4 páginas "Conceitos — X" fragmentadas para reconstituir a síntese que em A vive numa
só página.

**Leitura honesta:** isto não inverte os argumentos estruturais da Secção 2 (duplicação/deriva
semântica, manutenção — independentes de como o agente descobre ficheiros). Mas mostra que **a
alegação de eficiência de tokens/tempo da Secção 3 não generaliza para o cenário real de
descoberta** — nesse cenário, é C que sai mais barata e não mais lenta, e A paga um custo real
por o modelo não confiar inteiramente na página de conceito sintetizada sem verificar as fontes
primárias.

Isto aponta para uma ação concreta: a página de conceito canónica precisa de ser suficientemente
explícita e auto-suficiente (citações claras, uma frase do tipo "esta página é a fonte de
verdade sobre X") para que um agente a use com confiança sem reverificar os originais — o que
depende da qualidade do `llms.txt`, do schema de front matter (Eixo 5) e de como o `/query` real
(Secção 1 do `tasks.md`, ainda por escrever) é instruído a confiar no índice.

---

## 6. Decisão final

### 6.1 Decisão sob critério explícito: 1º coverage/correção, 2º custo de tokens/tempo

**Critério 1 — Coverage/correção: empate total** (Secção 4). Nas 108 respostas avaliadas, as 3
estruturas tiveram 100% de cobertura e 0 erros, sem exceção. Não desempata nada — passa-se ao
critério 2.

**Critério 2 — Tokens/tempo, medido no cenário que reflete produção real** (Secção 5, teste de
descoberta a frio, sem oráculo):

| Estrutura | Tokens (média) | Tempo (média) |
|---|---|---|
| A | 35 297 | 27 626 ms |
| B | 35 254 | 28 877 ms |
| **C** | **34 206** (mais barata, p ≤ 0,0001 vs. A e vs. B) | **26 622 ms** (mais rápida que B, p = 0,004; vs. A na fronteira, p = 0,047) |

**Decisão sob esta regra: Estrutura C.** O critério 1 empata; o critério 2 favorece C de forma
estatisticamente significativa em tokens e pelo menos não pior em tempo.

### 6.2 Fatores que ficam fora desta decisão de 2 critérios

Não entram na decisão acima porque não foram pedidos como critério, mas continuam documentados e
verdadeiros neste protótipo:

- **Risco de duplicação/deriva semântica** (Secção 2.1, 2.3, 4.4). Só A mantém cada conceito
  numa única página canónica; em B e C, os próprios subagentes auto-reportaram, sem serem
  perguntados, que a definição de SMKI estava duplicada — hoje idêntica, mas um risco real de
  divergência ao longo do tempo, sem um `lint` real a apanhar isso.
- **Esforço de manutenção ao ingerir** (Secção 2.1). A e C não exigem edição de config ao
  adicionar um PDF; B exige sempre uma nova secção manual.
- **A vantagem de tokens/tempo de A com oráculo é real, mas condicional** (Secção 3 vs. Secção
  5). Se o `/query` real for desenhado para confiar explicitamente na página de conceito sem a
  reverificar, A pode voltar a ser competitiva ou vencer em custo — isso ainda não foi testado
  com o comando real.

Se a equipa quiser incluir estes fatores como um 3º critério (ex.: "risco de manutenção a longo
prazo"), a decisão pode voltar a A. **Sob os 2 critérios exatamente como foram pedidos — coverage
primeiro, custo depois — a resposta é C.**

### 6.3 Refinamentos validados empiricamente (independentes de qual estrutura for escolhida)

1. **Secção 1 do `tasks.md` (o mais importante):** se a equipa optar por A apesar da decisão
   acima, o `/query` real deve ser explicitamente instruído a tratar `concepts/*.qmd` como fonte
   de verdade e a **não** reverificar automaticamente nos `documents/*.qmd` de origem — sem essa
   instrução, o custo de descoberta medido na Secção 5 anula a vantagem de custo de A. Testar
   depois de escrito (repetir o teste de descoberta com o `/query` real).
2. **Eixo 7:** a curadoria do `llms.txt` (Opção B) é necessária para dar ao agente confiança
   suficiente na página de conceito sem reverificar, qualquer que seja a estrutura escolhida.
3. **Eixo 4:** usar `sidebar: contents: auto: "<pasta>"` (não apenas `listing:`) para zero
   manutenção manual de navegação ao ingerir novos PDFs.
4. **Eixo 2 (Secção 4.2):** limitar o crescimento de uma página de conceito — se
   `concepts/<x>.qmd` acumular demasiadas facetas distintas de fontes diferentes (como aconteceu
   com `concepts/smki.qmd`, que passou a agregar 4 documentos), considerar subdividir em
   sub-secções com âncoras ou em páginas de conceito mais finas.

## 7. Ação seguinte

- Marcar a Secção 0 do [tasks.md](../tasks.md) como concluída. **Decisão sob os 2 critérios
  pedidos (coverage, depois custo): Estrutura C.** Se a equipa quiser pesar também duplicação/
  manutenção como 3º critério, rever antes de fechar — ver Secção 6.2.
- `_proto/` fica como evidência do protótipo; pode ser apagado depois de a equipa ratificar a
  decisão (não faz parte da wiki final em `wiki/`).
- Registar a decisão e os 4 refinamentos (instrução explícita no `/query` para confiar em
  `concepts/*.qmd` sem reverificar, curadoria obrigatória do `llms.txt`, sidebar `auto:`, limite
  de crescimento de páginas de conceito) no `CLAUDE.md` ao escrever a Secção 1.
- **Depois de escrever `/query` real (Secção 1):** repetir o teste de descoberta a frio (Secção
  5) contra o comando real (não um subagente genérico) para confirmar que a instrução de
  "confiar em `concepts/*.qmd`" resolve o custo de reverificação identificado.

---

## 8. Adenda — Schema v2 e `/query` real (2026-07-27)

Continuação do trabalho acima, feita depois de a Secção 6 ter fechado a decisão em C. Dois
desenvolvimentos motivaram este teste: (1) a extensão do schema de front matter com 4 campos de
verificação/proveniência — `status`, `confidence-score`, `supersedes`, `last-verified` (o padrão
referido como "LLM Wiki v2" nas Referências do README); e (2) a escrita real de `/query`
(`.claude/commands/query.md`) e `/lint`, que até aqui **não existiam** (a Secção 1.2 já tinha
assinalado isto como limitação). Ambos testados com o mesmo rigor estatístico do resto deste
documento (n=46/braço, `Workflow` em paralelo), na Estrutura A.

### 8.1 Schema v2 — fixtures corrigidos, efeito real e significativo

Corrigido removendo as pistas (título neutro sem versão ordinável, sem "revisto a partir de",
sem disclaimer redundante) — texto **idêntico** entre os dois corpora, só a metadata mudou.

**Cenário A — deteção de stale/supersedes:**

| | v2 (com schema) | v1 (controlo) | Teste |
|---|---|---|---|
| n | 44 | 45 | |
| Respondeu com o valor correto/atual (2 dias) | 16 (36,4%) | 0 (0%) | z=4,47, **p=0,0000080** |
| Respondeu com o valor stale (5 dias) | 7 (15,9%) | 40 (88,9%) | — |
| Outra resposta / sem número único | 21 (47,7%) | 5 (11,1%) | — |
| Assinalou conflito não resolvido | 35 (79,5%) | 13 (28,9%) | z=4,79, **p=0,0000016** |
| Mencionou substituição/versão mais recente | 44 (100%) | 6 (13,3%) | — |

**Cenário B — hedging por `confidence-score`:**

| | v2 (`confidence-score: 0.3`) | v1 (sem o campo) | Teste |
|---|---|---|---|
| n | 43 | 45 | |
| Deu o valor (7 dias) | 42 (97,7%) | 45 (100%) | — |
| Assinalou incerteza (hedged) | 43 (100%) | 40 (88,9%) | z=2,25, **p=0,024** |

**Interpretação:** a metadata v2 não resolve o conflito de forma limpa — torna os agentes **mais
cautelosos** (conflito não resolvido sobe de 29% para 80%). O efeito real e desejável é outro:
**sem metadata, 89% dos agentes usa confiantemente o valor stale; com metadata, isso cai para
16%.** O valor do schema está em impedir o uso confiante de informação desatualizada — trocando
falsa confiança por incerteza honesta (ou, nalguns casos, pela resposta certa) — não em garantir
sempre a resposta certa sozinho. O efeito em `confidence-score` é mais modesto mas real: fecha o
hiato de hedging de 89% para 100%. (184 corridas, 177 concluídas/7 erros, 5 472 974 tokens, ~7,2
min.)

### 8.2 `/query` e `/lint` reais escritos

Com base nos dois testes acima, `.claude/commands/query.md` e `.claude/commands/lint.md` deixam
de estar vazios. Regras principais do `/query`: visitar sempre a página de listagem da secção
(é lá que `status`/`confidence-score`/`last-verified` aparecem — a página individual não os
mostra); `supersedes` como critério autoritativo; nunca citar `status: stale`/`deprecated` sem
ressalva; `confidence-score` < 0,7 sempre verbalizado; e a regra de que **admitir conflito não
resolvido é o comportamento correto**, não uma falha (Secção 8.1 acima). O `/lint` valida o
schema (enum de `status`, range 0–1 de `confidence-score`, `supersedes` a apontar para ficheiro
existente) e acrescenta uma verificação nova: toda a página `stale` tem de estar referenciada por
um `supersedes`, e duas páginas não podem divergir sobre o mesmo facto sem uma relação
`supersedes` entre elas.

### 8.3 Teste do Refinamento #1 (Secção 6.3) — confiar em `concepts/*.qmd`

Testado isoladamente em Estrutura A, mesma pergunta ampla ("O que é o SMKI e como é usado?"),
com vs. sem a instrução explícita de confiar na página de síntese sem reverificar:

| | Com instrução | Sem instrução (baseline) | Teste |
|---|---|---|---|
| n | 46 | 46 | |
| Ficheiros lidos (média ± dp) | 2,59 ± 0,54 | 3,13 ± 0,62 | Welch t=-4,49, **p=0,0000072** |
| Reverificou nos `documents/*` depois da síntese | 0/46 (0%) | 46/46 (100%) | z=-9,59, **p≈0** |
| Cobertura completa (4 factos-chave) | 46/46 (100%) | 46/46 (100%) | empate |

**Confirma a hipótese por completo.** Sem a instrução, 100% dos agentes encontrava
`concepts/smki.llms.md` mas ia à mesma confirmar nos documentos de origem — exatamente o
comportamento descrito na Secção 5. Com a instrução, 0% o fez, sem perder cobertura. (92
corridas, 0 erros, 2 618 347 tokens, ~2,1 min.)

### 8.4 Re-teste A vs. B vs. C com `/query` real — a decisão da Secção 6.1 inverte-se

Repetição do teste de descoberta a frio da Secção 5, mesma pergunta ampla, agora com o `/query`
real (Secção 8.2, incluindo a instrução de confiança em `concepts/`) em vez de um subagente
genérico:

| Estrutura | n | Ficheiros lidos (média ± dp) | Cobertura completa |
|---|---|---|---|
| **A** | 46 | **3,04 ± 0,29** (min 3, max 5) | 46/46 (100%) |
| B | 46 | 8,93 ± 0,49 (min 8, max 10) | 46/46 (100%) |
| C | 46 | 6,00 ± 0,00 (sempre exatamente 6) | 46/46 (100%) |

| Comparação | Welch t | p (aprox. normal) |
|---|---|---|
| A vs. B | t=-69,87 | p≈0 |
| A vs. C | t=-68,00 | p≈0 |
| B vs. C | t=40,62 | p≈0 |

Sem sobreposição nenhuma entre as três distribuições. Comparando com a Secção 5 (subagente
genérico, sem `/query` real): **A caiu de 5,6±1,3 para 3,04±0,29 ficheiros** (~46% menos); C
manteve-se estável (5,0→6,0); B manteve-se pior (8,4±0,7→8,93±0,49).

**Aplicando a mesma regra de decisão da Secção 6.1 (1º coverage, 2º custo): coverage continua
empatado (100% nas três); o custo agora decide a favor de A, não de C.** A vantagem teórica de A
já prevista na Secção 3 (com oráculo) deixa de ser anulada pelo problema de "não confiar na
síntese" da Secção 5 — esse problema era do subagente genérico usado no teste, não de uma
limitação estrutural de A, e o `/query` real corrige-o. (138 corridas, 0 erros, 4 415 173
tokens, ~4,3 min.)

### 8.5 Limitações desta adenda

- **Ficheiros lidos, não tokens reais.** Ao contrário do resto deste documento (`subagent_tokens`
  reportado pela infraestrutura), aqui mede-se contagem de ficheiros — não há forma limpa de
  atribuir tokens por corrida individual quando dezenas de agentes correm em paralelo no mesmo
  `Workflow`. A relação ficheiros→tokens era quase linear no resto do documento, por isso é uma
  proxy razoável, mas não é o mesmo dado.
- **Só a pergunta ampla (Q1), não a bateria completa.** As Secções 4.2/4.3 testaram Q2–Q7 em
  cenários variados (conceito isolado, controlo sem sobreposição, facto estreito, anti-
  alucinação) — esta adenda não confirma se a vantagem de A se mantém nos cenários onde a Secção
  4.2 já tinha mostrado B a ganhar (Q3, Q4).
- **`_proto/structure-a-control` e os fixtures sintéticos** (`smki-interface-design-addendum.qmd`,
  `smki-response-time-inference.qmd`, `status: stale` em `smki-interface-design.qmd`) foram
  criados só para estes testes — não fazem parte do corpus real do SEC nem da wiki final. Podem
  ser removidos depois de a equipa ratificar as conclusões desta adenda.

### 8.6 O schema ajuda a reduzir custo, ou é só a instrução?

Pergunta de seguimento à Secção 8.4: o schema v2 contribui, por si, para a redução de ficheiros/
tokens vista nas Secções 8.3/8.4, ou é inteiramente um efeito da instrução do `/query`? Testado
isolando a variável: **mesma instrução** de confiar em `concepts/` nos dois braços, variando só
o corpus — `_proto/structure-a` (com schema) vs. `_proto/structure-a-control` (sem os 4 campos),
mesma pergunta ampla.

| | Com schema (v2) | Sem schema (v1) | Teste |
|---|---|---|---|
| n | 46 | 46 | |
| Ficheiros lidos (média ± dp) | 2,07 ± 0,25 (min 2, max 3) | 2,15 ± 0,51 (min 2, max 5) | Welch t=-1,03, p=0,303 (não significativo) |
| Citou `status`/`confidence-score` como justificação | 3 (6,5%) | 0 (impossível, campo inexistente) | z=1,76, p=0,078 (marginal) |
| Cobertura completa | 46/46 | 46/46 | empate |

**Resposta: não — o schema não reduz ficheiros/tokens além do que a instrução já reduz
sozinha.** A diferença de médias é pequena e não significativa (p=0,303). A instrução "confia na
síntese, não reverifiques" já leva o custo perto do mínimo possível (2 ficheiros: `llms.txt` +
a página de conceito) mesmo sem nenhum campo de verificação disponível, porque o texto da síntese
já cobre a pergunta por completo — não há nada que sinalize um problema, com ou sem schema.

Observação exploratória, não confirmada com teste de variância dedicado (F-test/Levene): a
variância é bem menor com schema (dp 0,25, máximo 3) do que sem (dp 0,51, máximo 5) — pode
indicar maior consistência (menos corridas a abrir ficheiros a mais "por precaução"), mas fica
como observação, não conclusão. (92 corridas, 0 erros, 2 544 279 tokens, ~2,5 min.)

**Os dois mecanismos são independentes, cada um resolve um problema diferente:**
- **Schema** (`status`/`confidence-score`/`supersedes`) → melhora correção e honestidade perante
  conflito/incerteza (Secção 8.1) — não afeta custo.
- **Instrução do `/query`** (confiar em `concepts/`) → reduz ficheiros/tokens lidos (Secções
  8.3/8.4) — não precisa do schema para isso, embora o schema lhe dê uma base concreta para
  justificar a confiança quando há um conflito real a verificar.

**Síntese:** a instrução reduz custo sempre, independentemente de haver problema ou não. O schema
**não "confirma que está tudo bem"** no caso bom — aí é irrelevante/ignorado (só citado em 6,5%
das corridas, Secção 8.6). O que o schema faz é funcionar como **rede de segurança para o caso
mau**: é o que impede a confiança cega precisamente quando confiar cegamente seria um erro —
página stale ou substituída (Secção 8.1: uso confiante do valor desatualizado cai de 89% para
16% com o schema presente). Sem o schema, a instrução "confia na síntese" seria uma aposta cega
que funciona bem no caso comum (síntese correta) mas não tem forma de detetar quando não está; o
schema é o que torna essa aposta segura, não o que a confirma.

### 8.7 Decisão atualizada

**Decisão: Estrutura A, com o schema v2** (`status`/`confidence-score`/`supersedes`/
`last-verified`). A decisão da Secção 6.1 (Estrutura C) fica revertida para Estrutura A, sob as
mesmas 2 regras de decisão pedidas (coverage, depois custo), agora medidas contra o `/query` real
em vez de um subagente genérico — condicional às limitações da Secção 8.5 (só Q1 testada com o
comando real; recomenda-se repetir Q2–Q7 antes de fechar definitivamente). O schema v2 fica
validado como extensão a manter, não porque reduza custo (Secção 8.6 mostra que não reduz), mas
porque melhora correção/honestidade perante conflitos (Secção 8.1) — a redução de custo vem
inteiramente da instrução explícita no `/query` (Secção 8.2). Os dois, juntos — Estrutura A +
schema v2 + `/query` real com as regras da Secção 8.2 — são a configuração recomendada.

### 8.8 Ação seguinte (atualiza a Secção 7)

- ~~Marcar a Secção 0 do `tasks.md` como concluída com Estrutura C~~ — **atualizado**: marcar com
  **Estrutura A**, com nota a apontar para esta adenda (Secção 8) como justificação.
- Transcrever para `CLAUDE.md`: a estrutura escolhida (A), o schema de front matter completo
  (campos originais + os 4 do schema v2), e um resumo das regras do `/query`/`/lint` já escritas
  em `.claude/commands/`.
- Antes de fechar definitivamente: repetir Q2–Q7 (Secções 4.2/4.3) com o `/query` real, para
  confirmar que a vantagem de A generaliza além da pergunta ampla de síntese.
- Limpar `_proto/structure-a-control` e os fixtures sintéticos de teste (Secção 8.5) depois de a
  equipa ratificar. `_proto/structure-a-control` também serviu de corpus de controlo para a
  Secção 8.6 — remover só depois de ratificar essa conclusão também.

---

## 9. Adenda 2 — Ingestão real do corpus, `/lint` real, e Cristalização no `/query` (2026-07-27)

Continuação direta da Secção 8: aquela adenda validou a decisão (Estrutura A + schema v2) e
escreveu `/query`/`/lint` reais, mas ainda contra o corpus de protótipo em `_proto/`. Esta secção
regista o passo que faltava — **`wiki/` (a wiki real) não existia no repositório antes desta
sessão**, apesar de o `CLAUDE.md` já descrever a estrutura como "promovida de `_proto/structure-a/`
em 2026-07-27" (a mesma data). Essa promoção era prospetiva/documental, não um facto do
repositório: só `_proto/` (com as 3 estruturas candidatas + os fixtures sintéticos da Secção 8)
existia em disco. Confirmado isto, foi pedido explicitamente para **ignorar `_proto/` por
completo** e construir `wiki/` do zero a partir dos 5 PDFs reais em `pdfs/` — não copiar nem
reutilizar o conteúdo já escrito em `_proto/structure-a/`, mesmo sendo o mesmo corpus.

### 9.1 Ingestão real dos 5 PDFs para `wiki/`

Construído do zero, independentemente de `_proto/`: esqueleto Quarto (`_quarto.yml` com
`llms-txt: true` e sidebar `auto:` — refinamentos da Secção 6.3/8.7 aplicados desde o início, não
descobertos de novo), `documents/index.qmd` e `concepts/index.qmd` com os campos do schema v2 na
listagem (`status`/`confidence-score`/`last-verified`, conforme a limitação identificada na
Secção 1 do `query.md` de que estes campos só são visíveis na listagem, não na página
individual), 5 páginas `documents/*.qmd` (Appendix M, N, P, AD, Schedule 8), 2 páginas
`concepts/*.qmd` (`smki.qmd`, agregando as 4 fontes que tocam SMKI — exatamente no limite da
"Regra de crescimento" da Secção 4.2/6.3; `duis-messaging.qmd`), e `glossary.qmd`. `quarto render
.` sem erros; `llms.txt` + 11 `.llms.md` gerados.

`confidence-score` atribuído de forma deliberadamente não uniforme: 0.85–0.9 para os 3 apêndices
SMKI (lidos por completo, documentos curtos); 0.75 para o DUIS (Appendix AD, ~350 páginas na
fonte) e 0.6 para a GBCS (Schedule 8, ~400 páginas) — para estes dois, só a introdução/estrutura
foi lida a fundo, com uma secção "Nota de confiança" própria em cada página a assinalar
explicitamente que secções específicas (catálogo de Service Requests do DUIS, Message Categories/
Use Cases da GBCS) não foram sintetizadas em detalhe. Isto operacionaliza ao vivo o achado da
Secção 8.1: `confidence-score < 0,7` tem de ser verbalizado, não só guardado no front matter.

### 9.2 `/lint` real corrido contra a wiki real — primeira vez sem fixtures sintéticos

Ao contrário da Secção 8 (onde `/lint`/`/query` foram testados contra corpora com problemas
**desenhados de propósito** para os disparar — `status: stale` sintético, fixtures de conflito),
esta foi a primeira corrida do `/lint` real contra conteúdo genuíno, sem nenhum problema
plantado. Resultado: **0 erros bloqueantes** nas 11 páginas (front matter, enums, ranges,
`supersedes` e links todos válidos); **1 aviso sistemático** (as 8 páginas de conteúdo omitem
`last-verified` — esperado, dado que nenhuma passou ainda de `status: draft` a `verified`); 0
conflitos semânticos não documentados; 0 páginas `stale` órfãs. Confirma que a ingestão ficou
internamente consistente desde a primeira escrita, não só validável em teoria.

### 9.3 Nova capacidade no `/query`: Cristalização (Passo 5)

**Motivação:** perguntas que exigem cruzar várias páginas produzem sínteses que, sem mecanismo de
persistência, se perdiam no fim de cada `/query` — a próxima pergunta parecida repetia do zero o
custo de descoberta/síntese medido nas Secções 3/5/8.3–8.4.

**Evolução do desenho, em 3 iterações dentro da mesma sessão:**

1. **v1 — proposta com confirmação.** `/query` passa a detetar quando a resposta cruzou mais do
   que uma página e a propor, no fim da resposta, criar uma página nova em `concepts/` (ou
   estender uma existente, respeitando a "Regra de crescimento" de ≤4 fontes) — mas nunca escreve
   sem confirmação explícita do utilizador.
2. **Reforço de anti-duplicação, antes de testar ao vivo.** Identificado um risco não coberto
   pelo `/lint` existente: a heurística de conflito da Secção 8.2/3.3 do `lint.md` só apanha
   páginas que **discordam** sobre o mesmo facto — duas cristalizações parecidas que **concordam**
   (produto de perguntas semelhantes em sessões sem memória uma da outra) passariam sem ser
   detetadas. Adicionado: (a) em `query.md`, uma verificação obrigatória do **corpo** (não só
   título/categorias) dos `concepts/*.qmd` sobrepostos antes de propor página nova; (b) em
   `lint.md`, uma heurística nova — duas páginas de conceito com `categories`/`related`
   sobrepostos que afirmem substancialmente os mesmos factos, sem se citarem uma à outra, é
   sinalizado como possível redundância a fundir.
3. **Teste ao vivo (1 corrida manual, não n=46).** Pergunta desenhada para exigir cruzar
   `concepts/smki.qmd` + `documents/smki-coc.qmd` + `documents/smki-repository-coc.qmd` +
   `documents/dcc-user-interface.qmd`, sem nenhuma página existente a cobrir a comparação
   (limites de capacidade SMKI vs. DUIS). O `/query` identificou corretamente que o lado DUIS não
   tinha dados de capacidade ingeridos (Secções 3.1/3.7 do documento de origem, assinaladas como
   não lidas exaustivamente na Secção 9.1) e reportou a lacuna em vez de a inventar — coerente com
   o comportamento anti-alucinação validado nas Secções 4.3/8.1. Resultado:
   `concepts/capacity-management.qmd`, criada com `confidence-score: 0.55` — mais baixo do que
   qualquer página-fonte individual, porque metade da comparação pedida é uma lacuna reconhecida,
   não só incerteza sobre as fontes lidas.
4. **v2 — gravação automática, sem pedir confirmação (desenho final).** Decisão do utilizador:
   remover por completo o passo de confirmação da v1. **O `/query` já não pergunta se deve
   guardar — guarda sempre, automaticamente, sempre que a condição de disparo se verifica** (a
   resposta cruzou mais do que uma página), e só depois informa o utilizador, numa nota curta no
   fim da resposta, de que a informação foi guardada e onde. Isto move toda a responsabilidade de
   evitar páginas redundantes para a verificação de duplicação do ponto 2 — deixou de haver um
   utilizador a travar a escrita antes de ela acontecer. `query.md` e `CLAUDE.md` foram
   atualizados para remover toda a linguagem de "nunca escrevas sem confirmação explícita do
   utilizador" que a v1 tinha.

### 9.4 Estado no fim desta sessão

- `wiki/` existe, reflete a decisão da Secção 8.7 (Estrutura A + schema v2) com o corpus real
  completo, e já contém uma página nascida de cristalização (`concepts/capacity-management.qmd`)
  a par das nascidas de ingestão — primeira confirmação prática de que `concepts/*.qmd` pode ter
  as duas origens, como o `CLAUDE.md` já previa.
- `.claude/commands/query.md` (Passo 5 — cristalização automática, **sem pedir confirmação ao
  utilizador**, só verificação de duplicação antes de escrever) e `.claude/commands/lint.md`
  (heurística de duplicação) atualizados; `CLAUDE.md` atualizado para descrever a cristalização
  como automática e sem confirmação no bullet do `/query`.
- `_proto/` não foi tocado nem lido para gerar conteúdo desta ingestão, conforme pedido — a
  sobreposição de tópicos com `_proto/structure-a/` (mesmos 5 PDFs) é coincidência de fonte, não
  reaproveitamento de texto.

### 9.5 Limitações / por fazer

- A cristalização automática só foi validada com **1 corrida manual**, não com o rigor estatístico
  (n=46/braço) do resto deste documento — antes de confiar nela em produção, falta medir, por
  exemplo, taxa de deteção de duplicados ao longo de várias perguntas parecidas, e taxa de falsos
  positivos da heurística nova do `/lint`.
- A ingestão do DUIS (`documents/dcc-user-interface.qmd`) e da GBCS (`documents/gb-companion.qmd`)
  ficou deliberadamente incompleta (`confidence-score` 0.75 e 0.6) — o catálogo de Service
  Requests do DUIS e a maior parte da GBCS (Message Categories, Use Cases) não foram sintetizados
  em detalhe; `concepts/capacity-management.qmd` já regista uma dessas lacunas como ação futura.
- Itens já identificados na Secção 8.8 e ainda pendentes, não afetados por esta sessão: repetir
  Q2–Q7 com o `/query` real; limpar `_proto/structure-a-control` e os fixtures sintéticos.

---

## 10. Adenda 3 — Benchmark de esforço (low/medium/high) no `/query` real, com verificação de qualidade

Depois de a Secção 9 confirmar a ingestão real e a cristalização automática, testámos uma
variável nova, nunca isolada nas Secções 3–8: o **nível de raciocínio (`effort`)** do Sonnet ao
correr o `/query` real, mantendo o modelo fixo. Pergunta: o custo extra de `medium`/`high` traduz-
se em melhor qualidade de resposta, ou é só mais tokens/tempo para o mesmo resultado?

### 10.1 Metodologia

- **Desenho:** 25 perguntas distintas (não uma só repetida), cada uma corrida 1× a `low`, 1× a
  `medium`, 1× a `high` — desenho **pareado** (75 corridas), não amostras independentes como nas
  Secções 3/5/8. Testes estatísticos: Friedman (omnibus, 3 grupos relacionados) + Wilcoxon signed-
  rank pairwise, com correção de Bonferroni (α=0,05/3=0,0167 por comparação).
- **Isolamento:** cada uma das 75 corridas usou a sua própria **cópia isolada da wiki real**
  (`cp -r wiki/` para 75 pastas em `scratchpad/bench-full/`), para que a cristalização automática
  (Secção 9.3) de uma corrida nunca contaminasse outra corrida da mesma pergunta a outro nível de
  esforço — sem isto, a 2ª/3ª corrida de uma pergunta poderia encontrar uma página já cristalizada
  pela 1ª e parecer artificialmente mais barata.
- **Execução sequencial, não paralela:** o `Workflow` não expõe tokens-por-chamada quando várias
  corridas partilham o mesmo `budget.spent()` em paralelo — só dá o total agregado. Corremos as 75
  chamadas **sequencialmente**, lendo `budget.spent()` antes/depois de cada uma para isolar o
  delta de tokens por corrida. Tempo por corrida: cada subagente correu `date +%s%3N` (Bash) antes
  e depois de responder, devolvendo os dois valores no output estruturado (schema forçado via
  `StructuredOutput`) — evita o bug de estado de shell entre chamadas separadas já registado na
  Secção 3.4.
- **Perguntas:** mistura deliberada de síntese ampla (Q1, Q8, Q9), perguntas de página única (Q2,
  Q3, Q10, Q12, Q13, Q17, Q20, Q22), detalhe factual estreito (Q4, Q5, Q11, Q14, Q15, Q19, Q21) e
  **5 perguntas desenhadas como armadilha de alucinação** — pedem um facto que se sabe, de
  antemão, que a wiki não tem: Q6 (critério exato Critical/Non-Critical), Q7 (mapeamento chave→
  canal da DUIS), Q18 (lista exata de browsers/SO), Q24 (Target Response Time exato), Q25
  (algoritmo criptográfico exato da GBCS).
- **Verificação de qualidade (novo, a pedido do utilizador):** as 74 respostas com sucesso foram
  avaliadas por **25 agentes avaliadores independentes** (um por pergunta), cada um recebendo só
  as respostas dessa pergunta (nunca a wiki) e um **checklist de gabarito escrito por mim** a
  partir do conteúdo real da wiki — não uma reverificação pela mesma fonte que gerou a resposta.
  Cada resposta foi classificada `pass`/`partial`/`fail` e `hallucinated: true/false`, com ênfase
  explícita nas 5 perguntas-armadilha (instrução: inventar um valor específico nessas conta como
  hallucination mesmo que soe plausível).

### 10.2 Resultado — custo (tokens e tempo)

| Esforço | Tokens (média, n=25) | Duração (média, n=24/25) |
|---|---|---|
| low | 3383 | 26,5s |
| medium | 4238 (+25%) | 40,8s (+57%) |
| high | 5662 (+67%) | 53,3s (+103%) |

Friedman: tokens χ²=18,24 p=0,00011; duração χ²=27,0 p=0,000001 (ambos altamente significativos).
Pares (Bonferroni α=0,0167): `medium` vs `high` e `low` vs `high` significativos em tokens
(p=0,00073 e p=0,0018; dz=0,64 e 0,71); `low` vs `medium` só tendência (p=0,032, não sobrevive à
correção). Em duração, `low` vs `medium` e `low` vs `high` são significativos (p=0,00011 e
p<0,000001; dz=0,92 e 1,18); `medium` vs `high` é só tendência (p=0,013, não sobrevive).
**1 corrida falhou a meio** (`q23 low`) — excluída do desenho pareado de duração (n=24), mantida
em tokens (n=25) porque o delta ainda ficou registado antes da falha.

**Ressalva de medição:** a soma dos 75 deltas de tokens dá ~332 mil, mas o `Workflow` reportou
3,75 milhões de `subagent_tokens` no total da corrida — `budget.spent()` só captura uma fração do
custo real por chamada (não inclui todo o raciocínio/tool-calls internos do subagente). A
comparação *relativa* entre níveis de esforço continua válida (mesma métrica em todas as 75
corridas), mas os valores absolutos por corrida estão subestimados.

### 10.3 Resultado — qualidade (novo)

| Esforço | n | Pass | Partial | Fail | Alucinado |
|---|---|---|---|---|---|
| low | 24* | 16 (67%) | 4 | 4 | 1 |
| medium | 25 | 19 (76%) | 4 | 2 | 1 |
| high | 25 | 22 (88%) | 2 | 1 | 0 |

*low tem só 24 avaliações porque `q23 low` falhou a meio (Secção 10.2) e não produziu resposta a avaliar.

**A qualidade parece escalar com o esforço, mas isso precisa de um teste formal — os pontos
percentuais sozinhos não bastam.** Testado com **Cochran's Q** (omnibus, dados binários
pareados — `pass=1` vs `partial`/`fail=0` — mesma pergunta nos 3 níveis, n=24, excluído `q23` que
falhou a `low`) seguido de **McNemar pairwise** com Bonferroni (α=0,05/3=0,0167):

| | low | medium | high |
|---|---|---|---|
| Accuracy (pass estrito) | 66,7% (16/24) | 75,0% (18/24) | 87,5% (21/24) |

- **Cochran's Q: Q=6,33, p=0,042** — diferença global nominalmente significativa entre os 3 níveis.
- **McNemar pairwise, nenhuma comparação sobrevive à correção de Bonferroni:** `low` vs `high`
  fica mais perto (p=0,0625; `high` corrige 5 perguntas que `low` errava, `low` não corrige
  nenhuma que `high` errasse — um padrão direcional 5–0 limpo, mas ainda insuficiente para n=24);
  `medium` vs `high` p=0,25; `low` vs `medium` p=0,625.

**Conclusão honesta:** há um sinal direcional consistente a favor de `high` (nunca perde para
`low` em nenhuma das 24 perguntas), e o omnibus sugere que a diferença entre os 3 níveis é real —
mas **nenhum par individual atinge significância corrigida**. Ao contrário de tokens/tempo
(Secção 10.2, onde n=24-25 já dava efeitos claramente significativos), accuracy é uma variável
binária que precisa de mais perguntas para o mesmo poder estatístico. Aplicando a regra de decisão
do resto deste documento ("1º coverage/correção, 2º custo"): o critério 1 aqui **não produz um
vencedor estatisticamente confirmado — é inconclusivo por falta de poder, não um empate limpo**.
Não dá para "justificar" a escolha de um nível de esforço com base em accuracy sozinha com o rigor
usado no resto do documento; dá para dizer que a tendência aponta para `high`, condicional a mais
dados.

**Achado central sobre alucinação:** das 5 perguntas-armadilha (Q6, Q7, Q18, Q24, Q25), **só 1 em
15 respostas alucinou** — `Q6 a low`, que afirmou categoricamente um "critério exato" (baseado
numa consequência/mecanismo real, o Pre-Command assinado) como se fosse o critério de
classificação Critical/Non-Critical em si, quando a wiki não o define. As respostas `Q6 medium` e
`Q6 high`, e as 12 restantes das outras 4 perguntas-armadilha em todos os níveis, admitiram
corretamente a lacuna sem inventar. Isto sugere que o comportamento anti-alucinação validado nas
Secções 4.3/8.1 é robusto à variação de esforço, com uma exceção pontual em `low`.

**Anomalia separada, não é falta de qualidade de conteúdo:** 3 das 75 respostas (`q4 low`, `q10
low`, `q9 medium`) devolveram literalmente o texto `"test"` no campo de resposta — não
alucinaram, simplesmente não responderam. Excluindo estas 3 (n=71), as taxas de `pass` sobem
ligeiramente (73%/79%/88%) mas o padrão `low < medium < high` mantém-se. A causa exata não foi
investigada (não há indício de que seja um problema do `/query` em si — pode ser um artefacto do
mecanismo de output estruturado forçado usado só para este benchmark); fica como item em aberto.

**Achado mais importante, e o mais autocrítico:** 3 perguntas (Q15, Q16, Q22) falharam ou ficaram
`partial` **nos 3 níveis de esforço, sem exceção** (Q20 quase — só `high` passou). Investigação
mostrou que isto **não é o `/query` a falhar** — é a minha própria síntese na Secção 9.1 que não
transcreveu esse detalhe:
- **Q22** ("definições de Portal/SMKI Portal"): a página `documents/smki-interface-design.qmd`
  só regista que a Secção Appendix H "define... Portal", sem reproduzir a definição em si —
  nenhum nível de esforço podia recuperar um facto que a síntese nunca transcreveu.
- **Q15** (bloqueio de conta): a síntese de `smki-repository-coc.qmd` menciona "verificação de
  identidade da ARO" na secção de bloqueio, mas só noutra secção da mesma página (lodging) é que
  liga essa verificação ao "ARO Nomination Form" — a ligação entre as duas secções não está
  explícita.
- **Q16** (Use Cases de firmware): a síntese comprime `CS05a`–`CS09` numa frase sem atribuir cada
  código à sua função exata — a informação exata simplesmente não foi sintetizada a esse nível de
  detalhe.

**Implicação:** o nível de esforço do `/query` não compensa uma síntese demasiado comprimida na
ingestão — é um teto de qualidade imposto pela wiki, não pelo agente de pergunta. Reforça a
recomendação já registada em `capacity-management.qmd` (Secção 9.5): a wiki tem lacunas de
ingestão conhecidas que nenhuma melhoria no `/query` resolve sozinha.

### 10.4 Síntese e recomendação

Subir de `low` para `high` custa **+67% tokens e +103% tempo — isto é estatisticamente
confirmado** (Secção 10.2, Friedman/Wilcoxon com correção). Em troca, observa-se **+21 p.p. de
taxa de pass e 1 alucinação a menos em 74**, mas essa diferença de qualidade **não passa no teste
estatístico pairwise corrigido** (Secção 10.3, Cochran's Q/McNemar) — só o omnibus é nominalmente
significativo, e o padrão direcional (5–0 a favor de `high` vs. `low`) é sugestivo mas não
confirmado com n=24. Isto é uma assimetria importante para não simplificar em falso:

- **O custo extra de `high` é certo; o ganho de qualidade é só uma tendência, não uma certeza
  estatística**, com a amostra atual (25 perguntas).
- Aplicando literalmente a regra de decisão do resto deste documento ("1º coverage/correção, 2º
  custo"): o critério 1 não decide (inconclusivo, não empatado) — o mais correto, sob esse
  critério, seria não decidir só com estes dados, e não simplesmente cair para o critério 2 (que
  favoreceria `low` pelo custo) sem reconhecer essa lacuna.
- Se a equipa tiver de escolher já, com a informação disponível: a direção dos dados (Cochran's Q
  nominal, 5–0 direcional em `low` vs `high`, 0 alucinações em `high` vs. 1 em `low`) é
  suficientemente consistente para justificar `high` em contextos de risco elevado (ex.: decisões
  de compliance), mas isso é uma escolha de apetite ao risco da equipa, não uma conclusão que este
  documento possa apresentar como estatisticamente provada.
- Independentemente do nível de esforço escolhido, **corrigir as lacunas de ingestão identificadas
  na Secção 10.3 (Q15, Q16, Q22)** traz mais qualidade do que subir o esforço — é um teto que o
  esforço não ultrapassa, e não exige mais nenhum dado para ser decidido.
- **Próximo passo para decidir accuracy com confiança:** repetir este desenho com uma amostra
  maior de perguntas (a Secção 10.5 estima quantas seriam precisas) antes de declarar `high`
  estatisticamente superior a `low` em qualidade.

### 10.5 Limitações

- Grelha de avaliação (`pass`/`partial`/`fail`) definida e aplicada por agentes avaliadores a
  partir de um checklist que eu escrevi — não há segunda opinião independente sobre os
  checklists em si (ao contrário da Secção 4, que comparou múltiplas estruturas com o mesmo
  checklist; aqui há só uma estrutura, e o checklist não foi testado contra avaliadores humanos).
- n=25 perguntas é uma amostra pequena para 5 categorias de pergunta (síntese, página única,
  factual estreito, armadilha, mistas) — insuficiente para comparar taxas de sucesso *por
  categoria* com significância estatística; os números da Secção 10.3 são agregados.
- As 3 anomalias "`test`" (Secção 10.3) não foram investigadas até à causa raiz.
- Esta adenda usa um desenho pareado (25 perguntas × 3 níveis) em vez do desenho de amostras
  independentes do resto do documento — os testes estatísticos (Friedman/Wilcoxon) não são
  diretamente comparáveis aos Mann-Whitney/Welch t das Secções 3/5/8.
- **Poder estatístico da comparação de accuracy `low` vs `high` (Secção 10.3):** com a taxa de
  pares discordantes observada (5/24 = 20,8%, todos a favor de `high`), uma análise de poder
  (teste de proporção a 1 amostra vs. 0,5, α=0,0167, poder=0,8) estima que seriam precisas
  **~49 perguntas no total** se a taxa real de discordância a favor de `high` for ~90% (perto do
  5/5 observado), ou **~102 perguntas** se for uma taxa mais conservadora de 80%. Ou seja, para
  confirmar estatisticamente (e não só sugerir) que `high` tem melhor accuracy do que `low`,
  este desenho precisaria de 2×–4× mais perguntas do que as 25 usadas aqui.

### 10.6 Decisão da equipa

**Decisão: usar Sonnet com `effort: high` para o `/query`**, com a *accuracy* como critério
decisivo — a equipa optou por não esperar pela amostra maior estimada na Secção 10.5 e decidir já
com os dados disponíveis.

É importante registar isto com o mesmo rigor do resto do documento: esta decisão **não se apoia
numa diferença de accuracy estatisticamente confirmada** (Secção 10.3 — nenhum par sobrevive à
correção de Bonferroni; `low` vs `high` fica em p=0,0625, o mais próximo mas ainda acima de
0,0167). Apoia-se, em vez disso, no conjunto de sinais direcionais e qualitativos que, juntos,
tornaram `high` a escolha da equipa apesar da não-significância formal:

- Cochran's Q omnibus nominalmente significativo (p=0,042) — há uma diferença real entre os 3
  níveis, mesmo que o par específico não isole essa diferença com confiança.
- Direção 100% consistente: em nenhuma das 24 perguntas pareadas `low` corrigiu algo que `high`
  errasse — só o inverso (5 vezes).
- O único caso de alucinação em 74 respostas (Secção 10.3) ocorreu em `low`, não em `high` ou
  `medium` — para o domínio regulatório desta wiki (SEC/SMKI/DCC), o custo de uma alucinação não
  detetada supera, para a equipa, o custo de +67% tokens/+103% tempo por resposta.
- As 3 perguntas com teto de qualidade (Q15, Q16, Q22, Secção 10.3) são independentes do nível de
  esforço e continuam a precisar de correção na ingestão — não pesam contra `high`.

**Isto é uma decisão de apetite ao risco da equipa perante um resultado estatisticamente
inconclusivo, não uma correção às Secções 10.3/10.4** — fica registada como tal para que, se a
amostra for alargada no futuro (Secção 10.5) e o resultado vier diferente, seja claro que a
decisão de hoje foi tomada conscientemente com essa incerteza, não com base numa significância que
não existe.

**Ação seguinte:** aplicar `effort: high` como omissão ao invocar `/query` (manualmente, ou em
qualquer automação futura que o chame) até haver dados que justifiquem revisitar esta escolha.
