# Tratamento de Dados — rascunho

**Propósito:** registrar operações de tratamento que podem ser demonstradas pelo código revisado.

**Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nesta data.

> **RASCUNHO:** todo conteúdo legal está sujeito à revisão por profissionais jurídicos qualificados em cada jurisdição.

## Operações observadas

| Dados/categoria | Operação visível | Evidência |
|---|---|---|
| Endereço e telefone de perfil | Sanitização e gravação em metadados do usuário | `frontend/includes/profile/grpms-profile.php:827-887` |
| Documentos KYC e validade | Validação de expiração, upload privado e armazenamento de referência/metadados | `frontend/includes/profile/grpms-profile.php:698-705`, `:780-801`, `:898-903`; `frontend/includes/profile/profile-kyc-files.php:25-33` |
| Pedido, cliente, valores, pagamento e comissões | Inserção em tabela de pedidos do marketplace | `backend/fastapi_app/services/checkout/order_service.py:261-281` |
| Dados de faturamento e entrega | Persistência dos campos de endereço; e-mail/telefone apenas no endereço de cobrança | `backend/fastapi_app/services/checkout/order_service.py:419-454` |
| Identificador de transação e taxa | Leitura de metadados WooPayments e gravação de taxa | `backend/fastapi_app/services/checkout/financial_fee_service.py:7-23`, `:37-49` |
| Elegibilidade de afiliado | Consulta papel de afiliado e status KYC | `backend/fastapi_app/services/checkout/checkout_service.py:6-25` |

## O que não é demonstrável

Finalidade jurídica, base legal, controlador/operador, instruções contratuais, suboperadores, localização, frequência, segurança efetiva, fluxos completos, consentimento, avaliação de impacto e transferências são **UNVERIFIED**. Os repositórios não provam a configuração dos ambientes, infraestrutura, plugins externos nem os dados reais de banco.

## Revisão necessária

Preencher o registro organizacional e contratual a partir de fontes verificáveis e revisão jurídica em cada mercado. Não usar este documento como declaração de conformidade ou instrução de tratamento aprovada.
