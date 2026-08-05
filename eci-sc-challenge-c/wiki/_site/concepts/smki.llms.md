# SMKI (Smart Meter Key Infrastructure)

smki

pki

interface

code-of-connection

## Síntese

A **SMKI (Smart Meter Key Infrastructure)** é a infraestrutura de PKI (Public Key Infrastructure) do SEC que emite e gere Certificados para Organisations e Devices na rede de smart metering do Reino Unido. Três apêndices do SEC cobrem, em conjunto, o desenho técnico, o código de ligação e o repositório da SMKI:

- **[SMKI Interface Design Specification (Appendix M)](../documents/smki-interface-design.llms.md)** — especifica os protocolos e estruturas técnicas das quatro interfaces (SMKI Portal via DCC Gateway Connection, SMKI Portal via Internet, Ad Hoc Device CSR Web Service, Batched Device CSR Web Service): formato PKCS#10 das CSRs, cifras TLS, schemas XML de request/response.
- **[SMKI Code of Connection (Appendix N)](../documents/smki-coc.llms.md)** — define quem pode ligar-se (só AROs com credenciais IKI), como se autenticam (Authentication Client software + Cryptographic Credential Token protegido por PIN) e os limites de capacidade/demand management de cada interface.
- **[SMKI Repository Code of Connection (Appendix P)](../documents/smki-repository-coc.llms.md)** — cobre um sistema relacionado mas distinto: o repositório onde Certificados e outra informação da SMKI são lodged/consultados, com o seu próprio mecanismo de autenticação (username/password) e gestão de procura (forecast + limite de 120%).

## Pontos-chave cross-documento

- **Interfaces vs. Repository são sistemas distintos** com autenticação diferente: as interfaces de emissão de Certificados (Portal/Ad Hoc/Batched) usam TLS mutuamente autenticado com Certificado IKI num Cryptographic Credential Token; o SMKI Repository Portal usa username/password tradicional (Appendix P, Secção 3.1).
- **Só AROs (Authorised Responsible Officers)** podem operar em nome de uma organização em qualquer uma das três frentes — credenciais emitidas segundo o processo da **SMKI RAPP** (referenciado, mas não incluído, nestes três apêndices).
- **Gestão de capacidade tem números concretos e específicos por canal**: 375.000 Batched CSRs/ dia (SMKI Services, Appendix N), 150 Ad Hoc CSRs/24h por Party (Appendix N), 120% da previsão declarada para o Repository (Appendix P) — não confundir os limites de um documento com os de outro.
- **DCC Gateway Connection é o caminho preferencial** em todas as três frentes; acesso via Internet (sem DCC Gateway Connection) existe como alternativa com fluxo de autenticação próprio em cada caso.
- **A GBCS (SEC Schedule 8) consome Certificados emitidos pela SMKI**, sem os emitir ela própria: a Secção 12 da [GB Companion Specification](../documents/gb-companion.llms.md) define requisitos que os Certificados (incl. Device Certificates e Certificados com `RemotePartyRole = root`/ `issuingAuthority`) têm de cumprir para serem processados por um Device — a emissão/gestão desses Certificados continua a ser definida pelos três apêndices SMKI acima, não pela GBCS.

**Nota de crescimento:** esta página agrega agora 4 fontes distintas (Appendix M, N, P e Schedule 8) — o limite indicado em `CLAUDE.md` antes de considerar subdividir. Uma 5ª fonte deve disparar a divisão em sub-secções com âncoras ou em páginas mais finas.

## Fonte

Síntese derivada de: [`documents/smki-interface-design.qmd`](../documents/smki-interface-design.llms.md), [`documents/smki-coc.qmd`](../documents/smki-coc.llms.md), [`documents/smki-repository-coc.qmd`](../documents/smki-repository-coc.llms.md), [`documents/gb-companion.qmd`](../documents/gb-companion.llms.md) (Secção 12, requisitos de Certificados).
