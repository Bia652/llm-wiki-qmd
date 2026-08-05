# DCC User Interface Specification (DUIS)

dcc

messaging

interface

security

pki

## Propósito

Nos termos da Secção H3, a DCC User Interface Specification (DUIS) especifica os detalhes técnicos da DCC User Interface: mecanismos, formatos, protocolos e outros detalhes técnicos necessários para os Users enviarem e receberem comunicações de/para a DCC (Secção H3.3).

Existem variações no tratamento de mensagens para Dispositivos SMETS1, descritas na cláusula 1.4.

## Estrutura do documento

1.  Introduction (propósito, termos definidos, variações SMETS1)
2.  The Interface (mecanismos de ligação, web services, tratamento de erros)
3.  Messages Sent Over The Interface (Service Request Matrix, controlo de acesso, operações criptográficas, formatos de Request/Response, Alerts, definições de Service Request)

## SMKI (Smart Meter Key Infrastructure)

A cláusula 3.3 (Key Cryptographic Operations) define como as mensagens DUIS XML são assinadas digitalmente (ECDSA sobre a curva P-256), com as chaves públicas certificadas ao abrigo da **Smart Meter Key Infrastructure (SMKI)**. Existem três tipos de chave privada distintos:

| Chave | Usada por | Propósito |
|----|----|----|
| XML User Role Signing Private Key | User | Assinar Service Requests e Signed Pre-Commands |
| DCC Transform Private Key | DCC | Assinar Service Responses com Pre-Commands |
| DCC Access Control Broker Private Key | DCC | Assinar DCC Alerts e outras Service Responses |

## Mensagens DUIS (Service Requests / Responses)

- **Device Requests (Critical / Non Critical)** — dirigidos a um Device.
- **Non-Device Requests** — não dirigidos a um Device.
- **Signed Pre-Commands** — comandos assinados para entrega local/futura.

Cada Service Request/Signed Pre-Command inclui um Common Object (Command Variant) que indica ao DCC como tratar a mensagem: transformar para assinatura, enviar ao Device, devolver para aplicação local (Hand Held Terminal), ou executar diretamente pela DCC.

## Glossário (inline)

| Termo | Definição |
|----|----|
| DUIS | DCC User Interface Specification |
| SMKI | Smart Meter Key Infrastructure |
| DCC Alert | Alerta gerado no DCC Total System (exceto Device Alert) enviado a Users relevantes |
| Device Alert | Comunicação contendo um Alert (definido pela GBCS) gerado por um Device, ou um SMETS1 Alert |
| Command Variant | Valor de um Common Object que indica como o DCC deve tratar a mensagem |
| GBCS | Great Britain Companion Specification |
| CPL | Central Products List |

## Fonte

PDF original: [SEC-Appendix-AD-DCC-User-Interface-Specification-v5.4](../../../pdfs/SEC-Appendix-AD-DCC-User-Interface-Specification-v5.4-6-November-2025.pdf)
