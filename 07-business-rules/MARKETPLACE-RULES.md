Propósito: registrar o ciclo de venda observado no código e as regras comerciais executadas pelos dois repositórios.

> **Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Ciclo da venda

1. **Produto publicado e elegível.** O backend aceita produto com `post_status=publish` e aprovação ausente ou `approved`; também rejeita estoque esgotado e produto desativado na validação de link de afiliado. `backend/fastapi_app/services/checkout/checkout_service.py:28-38, 45-77`
2. **Link de afiliado (se houver).** O link precisa estar ativo para a validação no checkout. O afiliado precisa ter papel `yith_affiliate` e KYC `approved`; a quantidade e desconto devem corresponder à configuração. `backend/fastapi_app/services/checkout/checkout_service.py:6-25, 67-89`
3. **Sincronização de pedido.** A importação só aceita `processing` ou `completed`, com itens identificados, conjunto de itens íntegro e dados consistentes com os itens WooCommerce. `backend/fastapi_app/services/checkout/order_service.py:42-83, 504-514`
4. **Registro financeiro.** O backend grava cada linha em `grpms_main_orders`, calcula taxas/comissões e inicializa status de comissão e saque como `pending`, além de datas pagas nulas. `backend/fastapi_app/services/checkout/order_service.py:250-283, 375-388`
5. **Entrega.** Item virtual ou baixável começa com `shipping_status=delivered`; item físico começa `pending`. `backend/fastapi_app/services/checkout/order_service.py:285-318`
6. **Conclusão e elegibilidade de pagamento.** A interface financeira filtra comissão para pedido `completed`, entrega `delivered`, sem data de pagamento, sem reembolso/remoção e com comissão diferente de zero. Uma solicitação só aparece como disponível depois de 15 dias da criação e se ainda não houver solicitação processada. `frontend/includes/financial/grpms-payment.php:187-237`

## Comissões e limites comprovados

A comissão inicial é calculada sobre o total da linha menos a parcela proporcional de `operations_fee`. O backend lê a comissão percentual da plataforma e a taxa do afiliado validado; no modo waterfall há cálculo em centavos e arredondamento half-up. `backend/fastapi_app/services/checkout/order_service.py:156-238`

**Liberação ao vendedor/afiliado:** a condição verificável para a fila de valores disponíveis na interface é simultaneamente: pedido `completed`; `shipping_status=delivered`; data de pagamento específica nula; sem reembolso (`is_refunded=0`); não removido (`is_trashed=0`); comissão não zero; e idade de 15 dias. Para vendedor, usa `vendor_paid_date`/`vendor_commission`; para afiliado, `affiliate_paid_date`/`affiliate_commission`. `frontend/includes/financial/grpms-payment.php:197-237`

Isso comprova elegibilidade/visibilidade para solicitar, não que o dinheiro foi transferido. A implementação de conclusão automática documenta que não faz liberação de payout e declara depender de aprovação de integração/schema. `backend/fastapi_app/services/checkout/auto_complete.py:1-7`. A operação de transferência efetivamente executada e suas condições finais são **UNVERIFIED** neste escopo.

## Exceções

Mudanças de estado de pedidos já sincronizados só aceitam `pending` ou `on-hold` → `processing`, e `pending`, `processing` ou `on-hold` → `completed`. Estado financeiro (reembolso, remoção, data paga ou item em payout) bloqueia transição. `backend/fastapi_app/services/checkout/order_service.py:16-20, 541-587`

Para limites e gaps, veja [BUSINESS-EXCEPTIONS.md](BUSINESS-EXCEPTIONS.md).
