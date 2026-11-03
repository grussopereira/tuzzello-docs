Propósito: documentar validação de afiliado, atribuição e condição de elegibilidade da comissão.

> **Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Validação para atribuição

Para validar link no checkout, o backend exige link com `link_status=active`, produto elegível/publicado, estoque diferente de `outofstock`, afiliado com papel `yith_affiliate` e `kyc_last_status=approved`, e quantidade/desconto válidos para o produto. `backend/fastapi_app/services/checkout/checkout_service.py:6-25, 28-38, 45-89`

Ao sincronizar o pedido, o backend verifica hash, afiliado e correspondência de produto/quantidade. A rotina de importação aceita link não marcado `deleted` para preservar crédito pós-venda; isso não significa que o link ainda possa ser usado para uma nova validação de checkout, que exige `active`. `backend/fastapi_app/services/checkout/order_service.py:99-127`

## Comissão

O backend salva comissão de afiliado na linha de pedido correspondente ao produto e quantidade atribuídos; calcula o valor a partir da taxa registrada no link e da base líquida depois da parcela de taxa operacional. A taxa e o cálculo são aplicados na importação financeira. `backend/fastapi_app/services/checkout/order_service.py:156-238`

Para entrar como disponível para solicitação, a comissão precisa satisfazer `completed`, `delivered`, `affiliate_paid_date IS NULL`, não reembolsada, não removida, valor diferente de zero e 15 dias desde `created`, além de não haver referência de solicitação processada. `frontend/includes/financial/grpms-payment.php:187-237`

## Visitas e conversão

O frontend associa visita ao pedido no evento `woocommerce_thankyou`, desde que haja hash/ID de afiliado e o pedido não esteja falho/cancelado; o restante do ciclo de visita é apresentado pelo dashboard. `frontend/includes/affiliation/grpms-affiliategeneral.php:127-152`

## Não verificado

Regras completas de inscrição/aprovação/desativação de afiliado e transferência efetiva de saldo são **UNVERIFIED**. KYC aprovado e papel YITH são gates de elegibilidade na validação de link, não prova de processo de onboarding integral.
