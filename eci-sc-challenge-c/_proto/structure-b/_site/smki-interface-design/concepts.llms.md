# Conceitos — SMKI Interface Design

security

pki

interface

Conceitos extraídos da [SMKI Interface Design Specification](../smki-interface-design/overview.llms.md).

## SMKI (Smart Meter Key Infrastructure)

A **Smart Meter Key Infrastructure (SMKI)** é a infraestrutura de chave pública (PKI) que certifica as chaves usadas para assinar digitalmente as comunicações no ecossistema SEC.

## Emissão de certificados

A DCC mantém o SMKI Service Interface através de três canais de submissão de Certificate Signing Requests (CSR):

| Interface | Tipo de CSR |
|----|----|
| SMKI Portal (via DCC Gateway Connection) | Organisation CSR e Device CSR (Ad Hoc ou Batched) |
| Ad Hoc Device CSR Web Service | Device CSR (ad hoc) |
| Batched Device CSR Web Service | Device CSR (batched) |
