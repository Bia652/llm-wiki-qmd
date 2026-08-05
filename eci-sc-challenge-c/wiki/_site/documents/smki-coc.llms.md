# SMKI Code of Connection (SEC Appendix N)

smki

pki

code-of-connection

dcc-gateway

authentication

## Propósito

Documento produzido nos termos da Secção L4.5 do SEC: define o mecanismo de ligação (connection mechanism), autenticação e gestão de procura (demand management) para as quatro interfaces de SMKI Services, complementando a especificação técnica das interfaces dada pelo SEC Appendix M (SMKI Interface Design Specification).

## Estrutura do documento

- **1. Connection Mechanism** — só Authorised Responsible Officers (AROs) com credenciais IKI emitidas segundo a SMKI RAPP podem acedes aos SMKI Services. Parties/RDPs com DCC Gateway Connection devem usá-la; sem ela, podem ligar-se via Internet.
  - **1.1 Browser Policy** — a DCC publica e mantém atualizada a lista de browsers/versões e sistemas operativos suportados para o SMKI Portal interface via DCC Gateway Connection.
- **2. SMKI Services interfaces** — as quatro interfaces (SMKI Portal via DCC Gateway Connection, Ad Hoc Device CSR Web Service, Batched Device CSR Web Service) permitem submeter Organisation CSRs, Ad Hoc Device CSRs e Batched CSRs e recuperar os Certificados correspondentes.
  - **2.3 Authentication** — o SMKI Portal via DCC Gateway Connection é protegido por sessão TLS mutuamente autenticada; requer instalação de “Authentication Client software” (assinado digitalmente, validado contra o Windows Trusted Root store) e um Cryptographic Credential Token protegido por PIN contendo o Certificado IKI e a chave privada da ARO. As Web Services (Ad Hoc e Batched) usam a mesma sessão TLS mutuamente autenticada com Certificado IKI próprio.
- **3. Managing Demand** — limites de capacidade:
  - Batched CSRs: processados overnight (20:00–08:00), capacidade total de **375.000 CSRs/dia** across todos os Authorised Subscribers; acima de **50.000 CSRs/24h** por Party requer aviso prévio de 7 dias ao Service Desk.
  - Ad Hoc Device CSRs via DCC Gateway Connection: limite de **150 CSRs/24h** por Party sem acordo prévio.
  - Ad Hoc Device CSR Web Service: **1 CSR por 0.8s** em horas core (07:00–20:00) e **1 CSR por 4s** fora desse período.
  - Batched Device CSR Web Service: resposta síncrona obrigatória antes de submeter novo Batched CSR; regras específicas de janela horária para acesso ao ficheiro de resposta (varia se submetido antes ou depois das 20:00).
- **Appendix A Definitions** — Ad Hoc Device CSR, Authentication Client, Batched CSR, One Way Authentication, SMKI User Guide.

## Fonte

[SEC-Appendix-N-SMKI-Code-of-Connection-v4.0.pdf](../../pdfs/SEC-Appendix-N-SMKI-Code-of-Connection-v4.0.pdf)
