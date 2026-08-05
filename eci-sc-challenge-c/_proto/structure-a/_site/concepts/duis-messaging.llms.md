# Mensagens DUIS (Service Requests / Responses)

dcc

messaging

Síntese do modelo de mensagens definido pela [DCC User Interface Specification](../documents/dcc-user-interface.llms.md).

## Tipos de pedido

- **Device Requests (Critical / Non Critical)** — dirigidos a um Device.
- **Non-Device Requests** — não dirigidos a um Device.
- **Signed Pre-Commands** — comandos assinados para entrega local/futura.

## Command Variant

Cada Service Request/Signed Pre-Command inclui um Common Object que indica ao DCC como tratar a mensagem: transformar para assinatura, enviar ao Device, devolver para aplicação local (Hand Held Terminal), ou executar diretamente pela DCC.

## Fontes

- [DCC User Interface Specification](../documents/dcc-user-interface.llms.md) — cláusulas 3.4–3.5
