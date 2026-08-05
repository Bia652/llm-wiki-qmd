# DCC User Interface Specification (DUIS)

dcc

messaging

interface

## Propósito

Nos termos da Secção H3, a DCC User Interface Specification (DUIS) especifica os detalhes técnicos da DCC User Interface: mecanismos, formatos, protocolos e outros detalhes técnicos necessários para os Users enviarem e receberem comunicações de/para a DCC (Secção H3.3).

Existem variações no tratamento de mensagens para Dispositivos SMETS1, descritas na cláusula 1.4.

## Estrutura do documento

1.  Introduction (propósito, termos definidos, variações SMETS1)
2.  The Interface (mecanismos de ligação, web services, tratamento de erros)
3.  Messages Sent Over The Interface (Service Request Matrix, controlo de acesso, operações criptográficas, formatos de Request/Response, Alerts, definições de Service Request)

## Segurança e criptografia

A secção 3.3 (Key Cryptographic Operations) define como as mensagens DUIS XML são assinadas digitalmente (ECDSA sobre a curva P-256), com as chaves públicas certificadas ao abrigo da **Smart Meter Key Infrastructure (SMKI)** — ver [SMKI](../concepts/smki.llms.md) para a síntese cross-documento deste conceito.

## Mensagens e Service Requests

Formatos de Service Request/Response, Command Variants e tipos de pedido (Device/Non-Device) — ver [Mensagens DUIS](../concepts/duis-messaging.llms.md).

## Fonte

PDF original: [SEC-Appendix-AD-DCC-User-Interface-Specification-v5.4](../../../pdfs/SEC-Appendix-AD-DCC-User-Interface-Specification-v5.4-6-November-2025.pdf)
