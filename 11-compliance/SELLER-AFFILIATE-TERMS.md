# Termos de Vendedores e Afiliados — rascunho

**Propósito:** resumir relações técnicas e fluxos visíveis entre vendedor, afiliado, produto e pedido.

**Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nesta data.

> **RASCUNHO:** todo conteúdo legal está sujeito à revisão por profissionais jurídicos qualificados em cada jurisdição.

## Comportamento técnico observado

O backend valida link de afiliado ativo e consulta elegibilidade do afiliado pelo papel e status KYC aprovado. `backend/fastapi_app/services/checkout/checkout_service.py:6-25`, `:67-77`.

Pedidos podem registrar vendedor, afiliado, método de pagamento, valores e comissões distribuídas. `backend/fastapi_app/services/checkout/order_service.py:261-281`.

A interface do vendedor apresenta relatórios de afiliados e fluxo de solicitação de cancelamento; a implementação de interface e mensagens está em `frontend/includes/seller/grpms-seller.php:145-210`, `:393-412`.

## Termos ainda não estabelecidos

O código não determina proposta/aceitação contratual, elegibilidade comercial completa, taxas acordadas, calendário de pagamento, impostos, reembolsos, chargebacks, propriedade intelectual, conteúdo proibido, resolução de disputas, suspensão, encerramento, autoridade e lei aplicável. Tudo é **UNVERIFIED** e requer elaboração/revisão jurídica por jurisdição antes de publicação ou uso como contrato.
