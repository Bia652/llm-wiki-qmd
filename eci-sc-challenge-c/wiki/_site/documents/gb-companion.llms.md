# Great Britain Companion Specification (SEC Schedule 8 / GBCS)

gbcs

security

messaging

dlms-cosem

zigbee

certificates

## Propósito

A **GBCS (Great Britain Companion Specification)**, SEC Schedule 8, é a especificação técnica que define a implementação única e consistente das Mensagens (Messages) trocadas entre Devices de smart metering (ESME, GSME, Communications Hub/CHF, PPMID, HCALCS, SAPC, IHDs) e entre esses Devices e organizações remotas (Remote Parties, ex.: Suppliers, Network Operators). Alinha-se com DLMS COSEM (Green/Blue Books) e ZigBee Smart Energy Profile (ZSE), e é referenciada pelas Technical Specifications de cada tipo de Device (GSMETS, ESMETS, CHTS, HCALCSTS, PPMIDTS, IHDTS, SAPCTS) como o documento que estas devem cumprir. Documento muito extenso e altamente técnico (~400 páginas); esta página sintetiza a estrutura de alto nível, sem reproduzir o detalhe normativo de cada Use Case/formato de mensagem.

## Estrutura do documento

- **1–2. Introduction / Structure** — GBCS é maioritariamente normativo (secções não marcadas “informative” são normativas); Devices no âmbito incluem ESME, GSME, CHF/Communications Hub, GPF, SAPC, PPMID, HCALCS e Type 2 Devices (ex.: IHDs).
- **3. Scope and Terminology** — define os conceitos centrais: uma **Message** é sempre `Command`, `Response`, `Alert` ou (só dentro do SMHAN) `HAN Only Message`; mensagens entre um Device e uma organização são **Remote Party Messages**; uma organização com credenciais já guardadas no Device é um **Known Remote Party (KRP)**, caso contrário é **Unknown Remote Party (URP)** (comandos de Response a um URP passam sempre pelo Access Control Broker do Device). Convenções notacionais (numeração hex/bin, endianness big/little, concatenação `||`, `GeneralizedTime`) também definidas aqui.
- **4. Security** — proteções criptográficas comuns a toda Message (integridade + autenticidade; Critical Messages acrescentam não-repúdio; certos campos exigem confidencialidade). Devices (exceto Type 2) têm pares de chave pública/privada e “Trust Anchor Cells” para guardar credenciais de Remote Parties. Um único **Originator Counter** por organização (não por Device) suporta a Protection Against Replay.
- **5–6. Message construction / Message Categories** — hierarquia de categorias de mensagem: `SME.C` (Command a um Device) com subcategorias `SME.C.C` (Critical), `SME.C.NC` (Non-Critical) e `SME.C.PPMID-GSME`; e `SME.A` (Alert de um Device) com subcategorias `SME.A.C`/`SME.A.NC`. Cada categoria acrescenta requisitos de proteção/verificação aos da categoria-mãe.
- **7–8. Message structure / Encryption of Attributes** — requisitos de construção de mensagem ao nível DLMS COSEM, ZSE e ASN.1; derivação de chaves e AAD/plaintext/ciphertext para acesso a atributos sensíveis via COSEM e ZSE.
- **9. Time Synchronisation and Future Dated Remote Party Messages**.
- **10. ZSE Implementation** — tunnels, interação GSME/GPF, Hand Held Terminal (HHT), requisitos Sub GHz, gestão do SMHAN, VWD.
- **11. Downloading firmware images to Devices** — Use Cases `CS05a`–`CS09` (distribuição e ativação de firmware para Communications Hub, ESME/GSME/HCALCS, PPMID).
- **12. Requirements for Certificates** — requisitos aplicáveis a todos os Certificados, a Organisation Certificates, a Certificados com `RemotePartyRole = root`/`issuingAuthority`, e a Device Certificates — ponto de contacto direto com a [SMKI](../concepts/smki.llms.md), que é quem emite estes Certificados.
- **13. Managing Security Credentials on Devices** — Use Cases `CS02a`–`CS02g` (fornecer, atualizar e emitir credenciais de segurança em Devices/Load Controllers), incl. Pair-wise Authorisation e backup/restore de log via GPF.
- **14. Apply Prepayment Top Up** — Use Cases `CS01a`/`CS01b` e variantes com intervenção do consumidor (código numérico no Device ou num PPMID); cálculo/verificação do dígito de verificação do UTRN.
- **15. Message Codes** / **16. Event / Alert Codes** — códigos únicos por Remote Party Message (Secção 3.2) e códigos de Event/Alert.
- **17. Remote Party Usage Rights** — direitos de acesso a atributos/métodos e a Use Cases por Remote Party.
- **18. Message Templates** — templates GBZ/ZSE e DLMS COSEM, instanciação ilustrativa e vetores de teste criptográficos.
- **19–20. Use Cases / Mapping Table**.
- **21. Glossary** — glossário próprio da GBCS (termos capitalizados definidos ao longo do documento remetem para aqui).
- **22–25. Annexes** — classe DLMS adicional (Annex 1); Annexes 2–3 intencionalmente em branco; uso do ZigBee na GBCS (Annex 4, informativo).

## Nota de confiança

Síntese estrutural baseada na leitura das Secções 1–4.2 (introdução, scope, terminologia, início de security) e nos títulos/organização do resto do documento (TOC). O detalhe normativo de Message Categories (Secção 6), formatos de mensagem (7–8, 18), Use Cases (11, 13, 14, 19) e o Glossário (21) **não foi lido exaustivamente** — confidence-score reflete isto. Perguntas sobre um Use Case, formato de mensagem ou código de erro específico devem ir à fonte primária.

## Fonte

[SEC-Schedule-8-GB-Companion-Specification-v4.4 1.pdf](../../pdfs/SEC-Schedule-8-GB-Companion-Specification-v4.4%201.pdf)
