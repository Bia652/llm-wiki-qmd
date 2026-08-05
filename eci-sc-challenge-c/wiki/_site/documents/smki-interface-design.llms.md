# SMKI Interface Design Specification (SEC Appendix M)

smki

pki

interface

dcc-gateway

## Propósito

Documento obrigatório sob a Secção L4 do SEC: especifica os detalhes técnicos das interfaces à SMKI (Smart Meter Key Infrastructure) disponíveis a Authorised Subscribers, incluindo protocolos, padrões técnicos e estruturas de mensagens. Cobre também a interpretação dos termos “sending” e “receipt” para efeitos de medição de Target Response Times (Secção L8.3 do SEC).

## Estrutura do documento

- **1. Introduction** — propósito do documento e definição de “sending”/“receipt” para cada uma das quatro vias de submissão (Ad Hoc Web Service, Batched Web Service, SMKI Portal via DCC Gateway Connection, Organisation CSR via Portal).
- **2. SMKI interfaces** — define as quatro interfaces disponibilizadas pela DCC:
  - **SMKI Portal interface** (via DCC Gateway Connection): mecanismo assíncrono baseado em browser para submissão de Organisation CSRs e Device CSRs (ad hoc ou batch) por Authorised Responsible Officers (AROs). Usa HTTPS com TLS mutuamente autenticado, conforme com WCAG v2 nível AA, e só é acessível via DCC Gateway Connection.
  - **Ad Hoc Device CSR Web Service interface** — submissão síncrona de Device CSRs individuais por sistemas automatizados do Authorised Subscriber, via DCC Gateway Connection.
  - **Batched Device CSR Web Service interface** — submissão de Batched CSRs (lotes) para Device Certificates, também via DCC Gateway Connection.
  - Todas as CSRs seguem o padrão **PKCS#10**; salvo notificação imediata de rejeição pelo Authorised Subscriber, um Certificado é considerado aceite (Secções L11.5–L11.7 do SEC).
- **Appendix A–B** — mensagens e schema XML da Ad Hoc Device CSR Web Service interface (exemplos de request/response, incl. erros como “Incorrect XML”).
- **Appendix C–E** — submissão e recuperação de resultados via Batched Device CSR Web Service interface (mensagens, schema XML, valores de `BatchStatus`/`Status`).
- **Appendix F** — estrutura das Certificate Signing Requests (Organisation CSR e Device CSR).
- **Appendix G** — credenciais de autenticação: TLS com cifras `ECDHE-RSA-AES256-GCM-SHA384` (e variantes), chaves de cliente/servidor RSA 2048 bit, hash SHA256; estrutura da CSR para Certificados IKI (client credentials) usados nas Web Services.
- **Appendix H** — definições de termos específicos deste apêndice (Ad Hoc/Batched Device CSR Web Service, ARO, CSR de Organização/Dispositivo, Portal).

## Fonte

[SEC-Appendix-M-SMKI-Interface-Design-Specification-v8.0.pdf](../../pdfs/SEC-Appendix-M-SMKI-Interface-Design-Specification-v8.0.pdf)
