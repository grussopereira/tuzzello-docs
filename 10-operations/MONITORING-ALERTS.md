Propósito: registrar sinais de monitoramento explicitamente presentes no código e lacunas de alertas operacionais.

Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

# Monitoramento e alertas

## Sinais localizados

- A rota de sincronização distingue reconciliação HTTP 409 e falha geral HTTP 500 (`backend/fastapi_app/routers/checkout/order_router.py:11-20`).
- A recepção do webhook WooCommerce registra status, razão, identificador de entrega e pedido; não incluir payload integral no diagnóstico (`backend/fastapi_app/routers/webhooks/woocommerce_order.py:65-80`).
- A sincronização de checkout escreve nota no pedido quando recebe resposta não 2xx (`frontend/includes/checkout/grpms-checkout.php:2038-2047`).
- O serviço de pedido registra condições de reconciliação (`backend/fastapi_app/services/checkout/order_service.py:23-25,81-83,115-126`).

## Alertas e limiares

Destino de logs, ferramenta de observabilidade, métricas, limiares, severidades, escalonamento e plantão: **UNVERIFIED**. Não há configuração operacional verificável nos trechos citados para definir alertas ou SLOs. Uma confirmação Stripe sem pedido ou comissão não deve ser classificada automaticamente por esse documento; integração correspondente: **UNVERIFIED**.

## Dados que não devem ser incluídos

Não incluir credenciais, tokens, conteúdo de `.env`, dados de cartão, payloads completos, nomes, e-mails, telefones ou endereços. Use identificadores mascarados quando compartilhar evidências.
