# Taxas e impostos

Propósito: descrever como taxas de pagamento e configurações de VAT aparecem no código.

Data: 2026-11-03. Descreve o HEAD de `frontend` (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e `backend` (`e75597fa`) nessa data.

Corrigido em 2026-11-04 conforme validação ARCH-FIN-VALIDATE (CHANGES REQUIRED).

Status: PARCIAL (afirmações não confirmadas em código estão marcadas NAO VERIFICADO).

## Taxas

`save_woopayment_fee_hpos` lê metadados WooPayments de ID da transação e `_wcpay_transaction_fee`, arredonda a taxa com `round(float(...), 2)` e grava `grpms_payment_fees` com método `card` e moeda literal `EUR`, usando upsert por chave duplicada (`backend/fastapi_app/services/checkout/financial_fee_service.py:3-48`). O import de pedido também procura `_wcpay_transaction_fee` e salva `operations_fee` antes de calcular comissões (`backend/fastapi_app/services/checkout/order_service.py:375-394`). A comissão aloca essa taxa proporcionalmente pelo total de linha (`order_service.py:178-189`).

## VAT configurável

A tela administrativa permite configurar percentuais para US, BR, ZA, PT e AO. Aceita campo vazio ou taxa de 0 a 100 com até duas casas e salva a opção `grpms_tax_liability_rates` (`frontend/includes/admin/tax_liability/grpms-tax-liability.php:8-27,30-59,79-87`).

NAO VERIFICADO: se esses percentuais são aplicados ao checkout, a regra de cálculo de VAT, incidência por produto/país, responsabilidade de recolhimento e relação com o campo de imposto do pedido. A tela, isoladamente, só comprova configuração.
