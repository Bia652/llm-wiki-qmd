---
description: Responde a uma pergunta usando exclusivamente a wiki (nunca os PDFs/artifacts originais), respeitando status/supersedes/confidence-score e citando as páginas usadas
argument-hint: [pergunta]
---

## Contexto

Vais responder à pergunta do utilizador usando **apenas a wiki** (páginas `.qmd`/`.llms.md`
já ingeridas), nunca os PDFs originais em `pdfs/`/`artifacts/` nem conhecimento teu próprio
sobre o domínio (SEC, DCC, SMKI, etc.). Se a wiki não tiver a resposta, di-lo — não completes
com conhecimento de fora da wiki nem inventes.

A wiki vive em `wiki/` (ver `CLAUDE.md`). Nunca uses `_proto/` — é só evidência de protótipo e
fixtures de teste, não faz parte da wiki real. Se o projeto Quarto estiver renderizado, usa o
site publicado (`wiki/_site/`, com `llms.txt` como índice de descoberta); caso contrário, lê
diretamente os `.qmd` em `wiki/`.

## Passo 1 — Descoberta

Começa pelo índice (`llms.txt` ou a página inicial da wiki) e decide, a partir daí, que páginas
vale a pena ler. **Não pares na primeira página que parecer responder** — visita sempre também
a página de listagem da secção correspondente (ex.: `documents/index.qmd`, `concepts/index.qmd`)
antes de dares a resposta como definitiva. É *só* na listagem que colunas como `status`,
`confidence-score` e `last-verified` costumam aparecer — a página individual normalmente só
mostra o título, categorias e o corpo do texto, não o front matter completo.

## Passo 2 — Confiar nas páginas de síntese (`concepts/`)

Quando o tema da pergunta tiver uma página de síntese cross-documento em `concepts/` (ou
equivalente na estrutura escolhida), trata-a como a fonte de verdade para esse tema e **não
percorras automaticamente todos os `documents/*.qmd` que ela cita só para "confirmar"**. A
página de síntese existe precisamente para evitar essa releitura repetida — reverificá-la sempre
contra as fontes primárias anula o ganho de custo de ter uma página de conceito dedicada.

Só volta às páginas `documents/*.qmd` de origem quando:
- a página de síntese estiver marcada `status: stale`/`draft`, ou tiver `confidence-score`
  baixo (< 0.7) na parte que cobre a resposta;
- a pergunta pedir um detalhe que a síntese não cobre explicitamente (ex.: uma cláusula exata,
  um valor que a síntese não menciona);
- houver um conflito sinalizado (ver Passo 3) que exija ir à fonte para desempatar.

Caso contrário, responde a partir da página de síntese e cita-a como fonte — não precisas de
abrir os documentos de origem "só para ter a certeza".

## Passo 3 — Resolver conflitos entre páginas (regras de prioridade, por esta ordem)

1. **`supersedes` é autoritativo.** Se a página X tem `supersedes: [Y]` no front matter, X
   prevalece sobre Y para qualquer facto sobreposto — mesmo que Y "pareça" mais oficial (ex.:
   ligada diretamente ao PDF original) ou X pareça mais provisória. Não uses outros sinais
   (nome do ficheiro, aparência de "documento principal") para contrariar isto.
2. **Nunca cites `status: stale` ou `status: deprecated` como facto atual sem ressalva.** Se
   encontrares uma página nesse estado, procura ativamente (na listagem da secção, ou no
   `related`/`supersedes` de outras páginas) qual a página que a substitui. Se não encontrares
   nenhuma, diz explicitamente que a informação pode estar desatualizada.
3. **`last-verified` mais recente não é, por si só, critério de desempate** — só conta combinado
   com `status`/`supersedes` explícitos (regra 1 e 2). Duas páginas com `status: verified` e
   datas diferentes, sem `supersedes` entre si, não são necessariamente conflituantes — podem só
   ter sido verificadas em alturas diferentes.
4. **`confidence-score` baixo (< 0.7) tem de aparecer na resposta.** Não apresentes um facto de
   baixa confiança com a mesma certeza que um facto verificado — diz que é uma estimativa/
   inferência, não uma citação direta de fonte primária.
5. **Se, mesmo aplicando 1–4 acima, ainda houver conflito genuíno sem forma de o resolver** (ex.: duas
   páginas com `status: verified`, sem `supersedes` entre si, a afirmar factos diferentes sobre o
   mesmo ponto), **di-lo explicitamente** em vez de escolheres uma arbitrariamente. Nomeia as duas
   fontes e o conflito. Isto é o comportamento correto, não uma falha — inventar uma escolha entre
   duas fontes igualmente válidas é pior do que admitir a ambiguidade.

## Passo 4 — Resposta

- Cita sempre a(s) página(s) da wiki usadas (caminho ou título).
- Se uma página estava `stale`/substituída, menciona-o (ex.: "segundo a versão atual, X; a versão
  anterior, agora desatualizada, dizia Y").
- Se sinalizaste baixa confiança ou conflito não resolvido, isso deve estar visível na resposta,
  não só na tua análise interna.
- Não repitas o texto integral das páginas — sintetiza e responde diretamente à pergunta.

## Passo 5 — Cristalização (guardar sínteses complexas)

Se a pergunta exigiu cruzar mais do que uma página (`documents/*.qmd` e/ou `concepts/*.qmd`)
para chegar à resposta — i.e., não foi uma leitura direta de uma única página de síntese já
existente — **guarda essa síntese automaticamente na wiki**, como página nova (ou secção nova de
uma página `concepts/*.qmd` existente), sem pedir confirmação. No fim da resposta, acrescenta uma
nota curta a informar o utilizador de que a informação foi guardada, por a pergunta ter exigido
cruzar múltiplas páginas — indica o caminho da página criada/atualizada (ex.: "Guardei esta
síntese em `concepts/<nome>.qmd`, por ter cruzado N páginas para responder.").

Como a gravação passa a ser automática, a verificação de duplicação abaixo é a única salvaguarda
contra páginas redundantes — sê rigoroso, já que já não há um utilizador a travar a escrita antes
de ela acontecer:

- Se a pergunta toca um `concepts/*.qmd` já existente e ele não passaria a ter >4 fontes depois
  de acrescentar esta síntese, atualiza essa página em vez de criar uma nova (ver "Regra de
  crescimento" em `CLAUDE.md`).
- **Antes de criar uma página nova, verifica duplicação, não só por título.** Lê o corpo (não só
  `title`/`categories` na listagem) de todos os `concepts/*.qmd` cujas `categories` ou `related`
  se sobrepõem, mesmo parcialmente, ao tema da pergunta — incluindo páginas que possam ter nascido
  de uma cristalização anterior, não só de um `/ingest`. Pergunta-te explicitamente: "já existe
  uma página que diz substancialmente isto?" antes de concluíres "não existe nenhuma página que
  cubra este cruzamento". Se encontrares uma síntese já muito próxima (mesmo com wording
  diferente), estende-a ou só a cites na resposta — não crias uma segunda página a dizer o mesmo.
- Só depois de confirmares que não há duplicação, se não existir nenhuma página de conceito que
  cubra este cruzamento, cria `concepts/<novo-conceito>.qmd`, com `related` a apontar para todas
  as páginas efetivamente usadas para montar a resposta.
- A página cristalizada segue o schema completo de `type: concept`: `status: draft` (é síntese
  ainda não revista independentemente), `confidence-score` honesto quanto à extrapolação feita
  para responder à pergunta (não só à fidelidade de cada fonte individual, que pode ser mais alta
  do que a confiança na resposta cruzada em si), `supersedes: []` salvo se substituir
  explicitamente outra página.
- Não cristalizes perguntas respondidas por leitura direta de uma única página — duplicaria
  conteúdo sem valor acrescido, e não há nota a acrescentar à resposta nesse caso.

Pergunta: $ARGUMENTS
