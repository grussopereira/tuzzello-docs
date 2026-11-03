# Regras de comissão

Propósito: registrar a fórmula e as salvaguardas de cálculo de comissão efetivamente presentes no backend.

Data: 2026-11-03. Descreve o HEAD de `frontend` (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e `backend` (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Base e fórmulas

`save_commissions` lê `_line_total` por item e aloca proporcionalmente `operations_fee` (`backend/fastapi_app/services/checkout/order_service.py:156-189`). Assim, a base é o total da linha informado pelo WooCommerce menos a parcela de `operations_fee`; o código não soma frete ou imposto nessa base. No modo waterfall, `GRPMS_COMMISSION_WATERFALL` precisa ser `1` ou `true`; o padrão é desligado (`order_service.py:133-142`).

No modo waterfall, cada taxa percentual é convertida em pontos-base por `Decimal`, multiplicação por 100 e quantização com `ROUND_HALF_UP` (`order_service.py:145-153`). Com `N` centavos líquidos, `P` pontos-base da plataforma e `A` pontos-base do afiliado:

- plataforma = `(N × P + 5000) // 10000`
- afiliado = `((N − plataforma) × A + 5000) // 10000`
- vendedor = `N − plataforma − afiliado`

A implementação, incluindo validação de inteiros não negativos e taxas até 10000 bps, está em `backend/fastapi_app/services/checkout/commission_math.py:4-11`. O afiliado é creditado no máximo uma vez e apenas quando produto e quantidade coincidem com o vínculo verificado (`order_service.py:196-203,221-234`).

Com o flag desligado, o código legado calcula `round(net × taxa / 100, 2)` para cada componente e zera comissão de afiliado que exceda o restante (`order_service.py:193-226`). Esse `round` é o builtin `round` do Python; no caso comum usa arredondamento para o par em empate.

## Exemplo derivado e teste

Entrada do teste: líquido `1995` centavos, plataforma `1250` bps e afiliado `750` bps. Saída esperada: plataforma `249`, afiliado `131`, vendedor `1615` centavos. Derivação: `(1995×1250+5000)//10000=249`; `((1995−249)×750+5000)//10000=131`; `1995−249−131=1615`. O teste afirma exatamente essa saída (`backend/tests/test_t1a_commissions.py:25-38`).

## Perguntas do proprietário

1. **Comissão sobre preço bruto ou líquido?** No waterfall, é sobre `_line_total` menos a alocação de `operations_fee`; o líquido é arredondado a centavos com `Decimal.quantize(Decimal("0.01"), ROUND_HALF_UP)` (`order_service.py:178-189`). O modo legado também subtrai a taxa operacional antes das comissões (`order_service.py:182-195`).
2. **Quem absorve as taxas Stripe?** **UNVERIFIED.** Nenhum trecho consultado estabelece atribuição de taxa Stripe. A taxa WooPayments `_wcpay_transaction_fee` é registrada em tabela auxiliar, e não há prova aqui sobre o responsável econômico (`backend/fastapi_app/services/checkout/financial_fee_service.py:16-48`).
3. **Como o desconto de afiliado interage?** **UNVERIFIED** quanto à aplicação/financiamento do desconto. A comissão usa `_line_total` já fornecido pelo WooCommerce; não há dedução ou adição separada de desconto nessa função (`order_service.py:160-189`).
4. **Regra de arredondamento?** Waterfall: função `compute_commissions`, inteiro centavos, fórmula `+5000 // 10000` (half-up para valores não negativos); base líquida e conversão de taxas usam `Decimal.quantize(..., ROUND_HALF_UP)` (`commission_math.py:4-11`; `order_service.py:145-153,183-189`). Legado: builtin Python `round` (`order_service.py:188-195,220-226`).
5. **Quem suporta perdas de chargeback?** **UNVERIFIED.** Não foi localizado tratamento de chargeback ou distribuição de perdas no cálculo consultado.
6. **O que ocorre com comissão já paga quando há reembolso?** **UNVERIFIED** para estorno de comissão paga. A guarda T2A registra `commission_released` se status não é `pending` ou se passou o prazo, e bloqueia elegibilidade; não há reversão de pagamento demonstrada (`backend/fastapi_app/services/checkout/refund_guard.py:74-76`; `refund_store.py:55-74`).
7. **Como se impede postagem duplicada?** Na sincronização de pedido, `GET_LOCK(grpms_sync_<order_id>, 10)` serializa tentativas, compara o conjunto de `item_id` existente com o esperado e retorna sem recalcular se já sincronizado (`order_service.py:504-545,589-595`). O trecho não prova chave única de comissão; uma constraint específica para comissão é **UNVERIFIED**.

## Lacunas

A execução depende da configuração de ambiente; o valor efetivo em produção é **UNVERIFIED**. Também é **UNVERIFIED** o vínculo contábil entre “desconto de afiliado” comercial e `link_row['commission']`, que é a taxa de comissão do afiliado.
