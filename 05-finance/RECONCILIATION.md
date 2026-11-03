# Reconciliação

Propósito: documentar os casos em que o código exige análise manual e as evidências que compara.

Data: 2026-11-03. Descreve o HEAD de `frontend` (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e `backend` (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Reconciliação de pedidos

O backend escreve aviso `affiliate_reconciliation` com ID do pedido, motivo e detalhes (`backend/fastapi_app/services/checkout/order_service.py:23-27`). Erros de vínculo, comissão, taxa negativa ou taxa superior à linha são enviados para reconciliação em pontos do cálculo (`order_service.py:99-128,174-192,203-225`). A rota de salvar pedido converte a condição em HTTP 409 “Order requires reconciliation” (`backend/fastapi_app/routers/checkout/order_router.py:11-18`).

O webhook compara ID, moeda, total pago, janela e estado financeiro do pedido WooCommerce contra registros locais; conflito recusa a operação (`backend/fastapi_app/services/checkout/wc_webhook.py:156-184`).

## Limites

UNVERIFIED: processo operacional de resolver a fila, reconciliação com extrato de adquirente, importação de transações externas e frequência de fechamento. Não foi executada consulta a banco nem chamada de provedor.
