Propósito: documentar webhooks de entrada, assinatura, idempotência e reprocessamento evidenciados no backend.

Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nesta data.

# Webhooks

## WooCommerce: pedido

Endpoint: `POST /webhooks/woocommerce/order`. O handler verifica se a integração está habilitada, limita o corpo a 1 MiB, lê os bytes sem transformação e delega a verificação ao serviço `services/checkout/wc_webhook.py`. Eventos aceitos são processados em threadpool e retornam `{"outcome": ...}`; eventos ignorados retornam o objeto produzido pelo verificador. (`backend/fastapi_app/routers/webhooks/woocommerce_order.py:37-72`.)

Erros observados incluem 404 quando desativado, 400 para `content-length` inválido, 413 para corpo grande e 503 se o armazenamento falhar. Falha de segredo é apresentada externamente como `unavailable`. (`woocommerce_order.py:42-60,73-82`.)

O segredo de webhook é `GRPMS_WC_WEBHOOK_SECRET`, nunca reproduzido aqui. A assinatura esperada é HMAC-SHA256 sobre os bytes originais do corpo, codificada em Base64; o header é `x-wc-webhook-signature`. A validação usa `hmac.compare_digest`. (`backend/fastapi_app/services/checkout/wc_webhook.py:52-71`.) O evento processado é `order.updated`, com status `processing` ou `completed`, pagamento e timestamps dentro das janelas verificadas; outros eventos podem ser ignorados. O payload também precisa conter ID, currency, total, transaction_id e refunds conforme validações do serviço. (`wc_webhook.py:72-111`.)

Há deduplicação transacional por ID de entrega e hash do payload; repetição consistente com estado concluído/noop/held responde `duplicate`. Conflito de ID/dados é recusado. Falhas são revertidas; recusas 422 são registradas para prevenir tentativas aceitas posteriormente com a mesma evidência. (`wc_webhook.py:126-155,195-220`.) Eventos com mais de 15 minutos ou mais de 2 minutos no futuro são ignorados (`wc_webhook.py:86-88`).

Reprocessamento manual: UNVERIFIED; o endpoint não expõe operação administrativa de replay. O retry de entrega externa é tratado como duplicata ou recusa persistida pela lógica descrita acima.

## EasyPost

`POST /api/webhooks/easypost` está registrado, mas responde 503 `Verified webhook integration is not configured`; não há processamento funcional de evento neste handler. Não configurar um webhook de produção com base apenas nessa rota. (`backend/fastapi_app/routers/checkout/easypost_router.py:190-192`.)

O esquema de assinatura, idempotência e reprocessamento de eventos EasyPost: UNVERIFIED / não implementado neste handler.
