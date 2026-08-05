# Conceitos — DCC User Interface

security

pki

dcc

messaging

Conceitos extraídos da [DCC User Interface Specification](../dcc-user-interface/overview.llms.md).

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
