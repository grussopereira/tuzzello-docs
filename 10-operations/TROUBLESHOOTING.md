Propósito: apoiar triagem de falhas observáveis no fluxo de checkout e sincronização descrito pelo código.

Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

# Solução de problemas

## Pedido não sincronizado

O checkout envia o pedido para `/api/checkout/save-order`; erros HTTP geram nota WooCommerce e a chamada bem-sucedida grava um transient (`frontend/includes/checkout/grpms-checkout.php:2029-2047`). O endpoint exige a dependência `require_wordpress`, converte falha de reconciliação em 409 e erro geral em 500 (`backend/fastapi_app/routers/checkout/order_router.py:8-20`). Verifique, por meios de observabilidade autorizados, o estado do pedido e a resposta HTTP. A disponibilidade e retenção desses logs são **UNVERIFIED**.

## Comissão não aparece

O backend insere linhas em `grpms_main_orders` e em seguida calcula o snapshot de comissões (`backend/fastapi_app/services/checkout/order_service.py:261-282,375-388`). A função consulta totais canônicos WooCommerce e atualiza `affiliate_id`, `item_total`, `affiliate_commission`, `vendor_commission`, `platform_commission` e `commission_amount` (`:156-173,228-234`). Confira os pré-requisitos descritos em [OPERATIONS-RUNBOOK.md](OPERATIONS-RUNBOOK.md). Não inferir payout ou disponibilidade a partir da mera presença de comissão.

## Pagamento Stripe confirmado, sistema sem atualização

Handler Stripe e vínculo entre confirmação do provedor e sincronização do pedido: **UNVERIFIED**. O código inspecionado usa o ciclo WooCommerce e API própria; não há evidência para recomendar consulta ou replay de evento Stripe. Siga o runbook apenas para as etapas de leitura e encaminhe a lacuna ao responsável técnico.

## Webhook indisponível

O webhook implementado é WooCommerce, default-off (`backend/fastapi_app/routers/webhooks/woocommerce_order.py:1,37-44`). Um HTTP 404 pode decorrer de estar desabilitado; o fluxo também limita tamanho e valida assinatura antes do processamento (`:44-68`). Não compartilhe assinatura, segredo ou corpo integral. Como ativar e como reentregar: **UNVERIFIED**.

## Conflito de dados de linha

`canonical_items` compara item, produto, quantidade e total da linha contra WooCommerce e sinaliza erro de integridade quando divergem (`backend/fastapi_app/services/checkout/order_service.py:42-83`). Preserve os registros e escale a reconciliação; procedimento de reparo manual: **UNVERIFIED**.
