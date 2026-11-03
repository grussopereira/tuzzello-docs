# Fluxo de pagamento

Propósito: descrever evidências de pagamento, sincronização e confirmação visíveis no código.

Data: 2026-11-03. Descreve o HEAD de `frontend` (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e `backend` (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Fluxo confirmado

O backend recebe `POST /webhooks/woocommerce/order`; o processador valida assinatura HMAC, tópico `order.updated`, status `processing` ou `completed`, `date_paid_gmt`, moeda, total, ID de transação e ausência de reembolsos no payload (`backend/fastapi_app/routers/webhooks/woocommerce_order.py:37-48`; `backend/fastapi_app/services/checkout/wc_webhook.py:52-110`). Eventos pagos são vinculados ao pedido WooCommerce armazenado, incluindo total e moeda, antes da conclusão (`wc_webhook.py:156-184`).

A confirmação grava evidência de pagamento e as datas de fim da janela de reembolso e liberação de payout como `paid + 7 dias` e `paid + 15 dias`; pedidos concluídos também geram item na outbox de status (`wc_webhook.py:185-199`). A sincronização de pedidos aceita apenas `processing` ou `completed` (`backend/fastapi_app/services/checkout/order_service.py:504-514`).

## Etapas não demonstradas

UNVERIFIED: autorização e captura no provedor. O webhook valida um pagamento já indicado pelo WooCommerce; o trecho não executa captura. UNVERIFIED: liquidação financeira no provedor, prazo de liquidação e fluxo completo de confirmação fora desse webhook. Não foi localizada implementação de Stripe nesse caminho.
