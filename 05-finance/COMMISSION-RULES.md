# Regras de comissão

Propósito: registrar a fórmula e as salvaguardas de cálculo de comissão efetivamente presentes no backend.

Data: 2026-11-03. Descreve o HEAD de `frontend` (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e `backend` (`e75597fa`) nessa data.

Corrigido em 2026-11-04 conforme validação ARCH-FIN-VALIDATE (CHANGES REQUIRED).

Status: PARCIAL (afirmações não confirmadas em código estão marcadas NAO VERIFICADO).

## Base e fórmulas

`save_commissions` lê `_line_total` por item e aloca proporcionalmente `operations_fee` (`backend/fastapi_app/services/checkout/order_service.py:156-189`). Assim, a base é o total da linha informado pelo WooCommerce menos a parcela de `operations_fee`; o código não soma frete ou imposto nessa base. No modo waterfall, `GRPMS_COMMISSION_WATERFALL` precisa ser `1` ou `true`; o padrão é desligado (`order_service.py:133-142`).

No modo waterfall, cada taxa percentual é convertida em pontos-base por `Decimal`, multiplicação por 100 e quantização com `ROUND_HALF_UP` (`order_service.py:145-153`). Com `N` centavos líquidos, `P` pontos-base da plataforma e `A` pontos-base do afiliado:

- plataforma = `(N × P + 5000) // 10000`
- afiliado = `((N − plataforma) × A + 5000) // 10000`
- vendedor = `N − plataforma − afiliado`

A implementação, incluindo validação de inteiros não negativos e taxas até 10000 bps, está em `backend/fastapi_app/services/checkout/commission_math.py:4-11`. O afiliado é creditado no máximo uma vez e apenas quando produto e quantidade coincidem com o vínculo verificado (`order_service.py:196-203,221-234`).

Com o flag desligado, o código legado calcula `round(net × taxa / 100, 2)` para cada componente e zera comissão de afiliado que exceda o restante (`order_service.py:193-226`). Esse `round` é o builtin `round(x, 2)` do Python aplicado a floats (`order_service.py:~187`), sem quantização Decimal; o modo legado usa round(x, 2) nativo do Python sobre float (semântica half-even sobre a representação binária, com erro de representação do float), diferente do ROUND_HALF_UP da cascata.

## Exemplo numérico derivado do código

Entrada: líquido `1995` centavos, plataforma `1250` bp, afiliado `750` bp (`compute_commissions`, `commission_math.py:9-11`).

- plataforma = `(1995 × 1250 + 5000) // 10000` = `(2493750 + 5000) // 10000` = `2498750 // 10000` = **249**
- afiliado = `((1995 − 249) × 750 + 5000) // 10000` = `(1746 × 750 + 5000) // 10000` = `(1309500 + 5000) // 10000` = `1314500 // 10000` = **131**
- vendedor = `1995 − 249 − 131` = **1615**

Referência de teste: `backend/tests/test_t1a_commissions.py:27` (caso `(1995, 1250, 750, (249, 131, 1615))`). O teste **não foi executado** nesta tarefa.

## Perguntas do proprietário

1. **Comissão sobre preço bruto ou líquido?** No waterfall, é sobre `_line_total` menos a alocação de `operations_fee`; o líquido é arredondado a centavos com `Decimal.quantize(Decimal("0.01"), ROUND_HALF_UP)` (`order_service.py:178-189`). O modo legado também subtrai a taxa operacional antes das comissões (`order_service.py:182-195`).
2. **Quem absorve a taxa de pagamento?** Comportamento do código: apenas quando `payment_method == "woocommerce_payments"`, `_wcpay_transaction_fee` (de `wp_wc_orders_meta`) é gravada como `operations_fee` em `grpms_main_orders` (`order_service.py:~375-390`). Em `save_commissions`, a parcela por item (`fee_share`, proporcional ao `_line_total`) é deduzida da base líquida antes da comissão (`order_service.py:~182-190`); como a comissão incide sobre essa base líquida, a taxa é dividida pro-rata entre plataforma, afiliado e vendedor. Outros meios de pagamento: nenhuma dedução de taxa (`operations_fee` permanece o valor existente, normalmente 0). A taxa é a taxa padrão de mercado da operação; não existe acordo comercial adicional (informado pelo Product Owner, 2026-11-04). A divisão acima é o comportamento do código.
3. **Como o desconto de afiliado interage?** `checkout_service.py:80-95` lê `grpms_meta.discount` (validado 0–100) e o devolve em `discount` do link. `save_commissions` **não usa** esse valor (`order_service.py:156-189` usa só `_line_total`). **Se o desconto reduz a linha do WooCommerce é NAO VERIFICADO.**
4. **Regra de arredondamento?** Waterfall: inteiros em centavos, half-up (`+5000 // 10000`) em `compute_commissions` (`commission_math.py:9-11`); taxa percentual → bp por `Decimal.quantize(1, ROUND_HALF_UP)` (`order_service.py:145-153`); líquido por item com `ROUND_HALF_UP` a 0,01 (`order_service.py:~183-185`). `fee_share` é float **sem arredondamento** (`order_service.py:~182`). Legado: builtin `round(x, 2)` sobre floats (`order_service.py:~187`); o modo legado usa round(x, 2) nativo do Python sobre float (semântica half-even sobre a representação binária, com erro de representação do float), diferente do ROUND_HALF_UP da cascata.
5. **Quem suporta perdas de chargeback?** **NAO VERIFICADO.** Não há código de chargeback em `fastapi_app`.
6. **O que ocorre com comissão já paga quando há reembolso (clawback)?** **NAO VERIFICADO.** Não há código de estorno/clawback de comissão paga em `fastapi_app`. Fatos confirmados: `refund_guard` (`refund_guard.py:49-81`) bloqueia elegibilidade com razões como `commission_released` (status ≠ `pending` ou ≥ 15 dias da criação), `payout_item_exists`, `window_expired` (≥ 7 dias após conclusão); `refund_store.py:1-5` se declara "Unwired T2A SQL helpers" (não ligado ao fluxo).
7. **Como se impede postagem duplicada?** Salvaguardas confirmadas em código:
   - `GET_LOCK('grpms_sync_<order_id>', 10)` e comparação do conjunto de `item_id` existente vs. esperado; retorna "already synchronized" se igual (`order_service.py:~504-545,589-595`).
   - errno 1062 (violação de unique) em `_insert_order_into_main_tables` → `OrderReconciliationError` (`order_service.py:~364-372`), mapeado para HTTP 409 em `routers/checkout/order_router.py:16-17`.
   - Webhook: PK de `delivery_id` + hash `payload_sha256`; duplicata retorna `duplicate` (`wc_webhook.py:~136-155`).
   - Outbox: `UNIQUE uq_wc_outbox_order_target (order_id, target_status)` (`sql/auto_complete/002_outbox_and_windows.sql:25`).
   - Payout: `UNIQUE uq_grpms_payouts_operator_transaction (operator_transaction_id)` (`sql/t07/003_unique_operator_transaction_id.sql:11`).
   - `uq_order_item` (`sql/b_t6/002_swap_unique_order_product_to_order_item.sql`) **NÃO FOI EXECUTADO**; requer aprovação do DBA. Se está aplicado em produção é **NAO VERIFICADO**.

## Lacunas

A execução depende da configuração de ambiente; o valor efetivo em produção é **NAO VERIFICADO**. Também é **NAO VERIFICADO** o vínculo contábil entre “desconto de afiliado” comercial e `link_row['commission']`, que é a taxa de comissão do afiliado.
