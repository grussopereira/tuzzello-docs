# Registro financeiro

Propósito: mapear registros financeiros gravados e limites do que se pode inferir deles.

Data: 2026-11-03. Descreve o HEAD de `frontend` (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e `backend` (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Registros encontrados

- `grpms_main_orders` recebe campos de comissão por item (`item_total`, `affiliate_commission`, `vendor_commission`, `platform_commission`, `commission_amount`) durante o snapshot inicial (`backend/fastapi_app/services/checkout/order_service.py:228-234`).
- `grpms_payment_fees` guarda a taxa e identificador de transação WooPayments (`backend/fastapi_app/services/checkout/financial_fee_service.py:37-48`).
- O webhook registra delivery, hash SHA-256 do payload, tópico, hora, resultado e identificador da transação; janela de pagamento guarda confirmação e datas derivadas (`backend/fastapi_app/services/checkout/wc_webhook.py:136-155,185-199`).
- A área de payout relaciona cabeçalho `grpms_payouts` e itens `grpms_payout_items`, com valores bruto, tarifa de operador, líquido, status, horário, operador, ID de transação e IDs de ator (`frontend/includes/financial/grpms-payment.php:1625-1650,2584-2618`).
- Helpers de reembolso T2A inserem operação rejeitada e evento de auditoria; o módulo declara que não está integrado ao fluxo, e a operação de reembolso completo não existe (`backend/fastapi_app/services/checkout/refund_store.py:1-5,83-90,125-139`).

UNVERIFIED: existência/instalação efetiva das tabelas no banco atual, completude do ledger contábil, lançamentos de chargeback, conciliação de saldos e trilha imutável para todos os ajustes. Nenhum banco foi consultado.
