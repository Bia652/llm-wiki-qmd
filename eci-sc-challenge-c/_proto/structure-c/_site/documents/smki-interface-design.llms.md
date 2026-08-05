# SMKI Interface Design Specification

security

pki

interface

## Propósito

A Secção L4 do Code estabelece a obrigação da DCC de manter o SMKI Service Interface de acordo com a SMKI Interface Design Specification. A Secção L4.4 define o conteúdo desta especificação, incluindo os protocolos e standards técnicos e os detalhes técnicos das interfaces aos SMKI Services relevantes para Authorised Subscribers.

## Estrutura do documento

1.  Introduction (propósito, Target Response Times)
2.  SMKI interfaces — SMKI Portal, Ad Hoc Device CSR Web Service, Batched Device CSR Web Service

## SMKI (Smart Meter Key Infrastructure)

A **Smart Meter Key Infrastructure (SMKI)** é a infraestrutura de chave pública (PKI) que certifica as chaves usadas para assinar digitalmente as comunicações no ecossistema SEC.

## Emissão de certificados

A DCC mantém o SMKI Service Interface através de três canais de submissão de Certificate Signing Requests (CSR):

| Interface | Tipo de CSR |
|----|----|
| SMKI Portal (via DCC Gateway Connection) | Organisation CSR e Device CSR (Ad Hoc ou Batched) |
| Ad Hoc Device CSR Web Service | Device CSR (ad hoc) |
| Batched Device CSR Web Service | Device CSR (batched) |

## Glossário (inline)

| Termo | Definição                             |
|-------|---------------------------------------|
| DUIS  | DCC User Interface Specification      |
| SMKI  | Smart Meter Key Infrastructure        |
| GBCS  | Great Britain Companion Specification |
| CPL   | Central Products List                 |

## Fonte

PDF original: [SEC-Appendix-M-SMKI-Interface-Design-Specification-v8.0](../../../pdfs/SEC-Appendix-M-SMKI-Interface-Design-Specification-v8.0.pdf)
