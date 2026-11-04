# Payouts

Propósito: descrever registro e auditoria de pagamentos manuais de comissões.

Data: 2026-11-03. Descreve o HEAD de `frontend` (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e `backend` (`e75597fa`) nessa data.

Corrigido em 2026-11-04 conforme validação ARCH-FIN-VALIDATE (CHANGES REQUIRED).

Status: PARCIAL (afirmações não confirmadas em código estão marcadas NAO VERIFICADO).

## Fluxo observado

A rotina de payout manual grava cabeçalho em `grpms_payouts` com operador, ID de transação, status `manual_confirmed`, data de pagamento e administrador; grava itens associados em `grpms_payout_items` (`frontend/includes/financial/grpms-payment.php:1625-1650`). A interface de auditoria apresenta valores bruto, taxa do operador, líquido, transação, administrador, status e itens (`grpms-payment.php:2584-2618`).

A função de conclusão toma lock nomeado derivado do ID de transação, valida reuso e usa verificação de elegibilidade antes de registrar (`grpms-payment.php:1509-1518,1702-1720,1823`). O backend também consulta itens de payout para impedir alteração de estado de pedido financeiramente comprometido (`backend/fastapi_app/services/checkout/order_service.py:541-560`).

NAO VERIFICADO: cronograma operacional de liberação, cálculo geral do saldo disponível, processamento automático pelo PayPal, tarifas incidentes e falha/estorno do operador. O código consultado documenta registro de payout manual.
