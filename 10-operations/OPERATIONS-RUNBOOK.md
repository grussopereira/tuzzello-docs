Propósito: orientar diagnóstico operacional de pedidos, pagamentos e comissões com base no código disponível.

Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

# Runbook operacional

## Pedido ou comissão ausente após pagamento Stripe confirmado

**Limite de evidência:** não foi localizado handler Stripe nos códigos examinados. O checkout chama a API do backend a partir do fluxo WooCommerce (`frontend/includes/checkout/grpms-checkout.php:1970-2048`); o backend grava e calcula comissões no endpoint `/api/checkout/save-order` (`backend/fastapi_app/routers/checkout/order_router.py:8-20`). A associação entre confirmação Stripe e esse fluxo é **UNVERIFIED**. O webhook presente é de WooCommerce, assinado e desabilitado por padrão (`backend/fastapi_app/routers/webhooks/woocommerce_order.py:1,37-44`); não o trate como webhook Stripe.

### Coleta de evidências

Use somente uma ferramenta de leitura aprovada para consultar os nomes de tabelas e campos abaixo. Não execute alterações, reprocessamentos ou consultas que revelem dados pessoais, cartões, credenciais ou valores sensíveis. Correlacione usando identificador interno de pedido autorizado e mascare-o em registros compartilhados.

1. No WooCommerce, confira `wp_posts.ID`, `wp_posts.post_type`, `wp_posts.post_status` e `wp_postmeta.post_id`, `wp_postmeta.meta_key`, `wp_postmeta.meta_value` para existência/status do pedido e metadados relevantes. O uso de armazenamento HPOS pode alterar onde o pedido reside; configuração e fonte operacional do pedido são **UNVERIFIED**.
2. Confira `wp_woocommerce_order_items.order_id`, `wp_woocommerce_order_items.order_item_id`, `wp_woocommerce_order_items.order_item_type`; e `wp_woocommerce_order_itemmeta.order_item_id`, `wp_woocommerce_order_itemmeta.meta_key`, `wp_woocommerce_order_itemmeta.meta_value`. O serviço valida itens, produto, quantidade e total de linha contra esses registros (`backend/fastapi_app/services/checkout/order_service.py:42-83`).
3. Confira `grpms_main_orders.order_id`, `grpms_main_orders.item_id`, `grpms_main_orders.order_status`, `grpms_main_orders.payment_method`, `grpms_main_orders.item_total`, `grpms_main_orders.commission_amount`, `grpms_main_orders.platform_commission`, `grpms_main_orders.affiliate_commission`, `grpms_main_orders.vendor_commission`, `grpms_main_orders.affiliate_id`, `grpms_main_orders.commission_status`. O insert e os campos financeiros constam em `backend/fastapi_app/services/checkout/order_service.py:261-282,329-358`; a atualização de snapshots está em `:228-234`.
4. Para validação de afiliado, confira `grpms_affiliate_links.hash`, `grpms_affiliate_links.affiliate_id`, `grpms_affiliate_links.product_id`, `grpms_affiliate_links.quantity`, `grpms_affiliate_links.commission`, `grpms_affiliate_links.link_status`; e `wp_options.option_name`, `wp_options.option_value`. O código verifica vínculo e taxa de plataforma (`backend/fastapi_app/services/checkout/order_service.py:86-96,99-127`). Não exponha hash nem valores pessoais em relatórios.
5. Para a exceção de taxa, confira `wp_wc_orders_meta.order_id`, `wp_wc_orders_meta.meta_key`, `wp_wc_orders_meta.meta_value` somente se `payment_method` for `woocommerce_payments`; a leitura de `_wcpay_transaction_fee` está em `backend/fastapi_app/services/checkout/order_service.py:375-387`. Isso não demonstra integração Stripe direta.

### Decisões

- **Pedido existe no WooCommerce, mas não em `grpms_main_orders`:** confira o caminho de chamada e resultado HTTP do POST de sincronização. O frontend envia a `/api/checkout/save-order`, marca transient apenas para HTTP 2xx e adiciona nota ao pedido em falha (`frontend/includes/checkout/grpms-checkout.php:2029-2047`). O endpoint retorna 409 para reconciliação e 500 para outras falhas (`backend/fastapi_app/routers/checkout/order_router.py:11-20`). Logs/observabilidade disponíveis para cada ambiente: **UNVERIFIED**.
- **Pedido aparece, comissão zerada/ausente:** confirme se os itens WooCommerce passam pela validação, se `grpms_platform_commission` tem valor finito entre 0 e 100, e se os itens importados correspondem ao `item_id`. Código: `order_service.py:42-95,156-173,196-234`. Se for necessário decidir sobre elegibilidade/atribuição afiliada, valide vínculo e elegibilidade em código (`:99-127`); não presuma que a comissão de afiliado se aplica a todos os itens.
- **Estado permanece `pending`, `failed` ou diverge do pagamento:** não altere estado manualmente com base apenas na confirmação Stripe. As regras de transição de status dependem do caminho de sincronização e marcador verificado (`order_service.py:16-20`); vínculo desse marcador com Stripe é **UNVERIFIED**. Encaminhe para responsável técnico com referências de pedido mascaradas e evidências de status.
- **Webhook recebido ou erro de entrega:** o endpoint WooCommerce `/webhooks/woocommerce/order` retorna 404 quando desabilitado e valida evento/carga antes de processar (`woocommerce_order.py:37-68`). Não habilite nem reenvie como procedimento operacional sem instrução verificada: **UNVERIFIED**.
- **Qualquer divergência financeira, conflito ou falta de linha canônica:** pare a correção manual e encaminhe para reconciliação técnica. O backend diferencia conflito 409 e erro 500; retry automático, janela e procedimento de replay são **UNVERIFIED**.

### Restrições deste runbook

Não contém comandos SQL executáveis nem autoriza comandos contra banco. Procedimentos de console do Stripe, WooCommerce, infraestrutura, retry, replay, edição manual ou contato com provedores são **UNVERIFIED** e precisam de fonte operacional aprovada.
