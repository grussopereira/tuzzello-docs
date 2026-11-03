Propósito: delimitar as integrações externas que participam dos fluxos API e marketplace.

Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nesta data.

# Integrações

## WordPress e plugin Tuzzello

O plugin PHP compõe a experiência do marketplace, guarda/usa sessão do WordPress e chama a API; o backend confia no segredo compartilhado para rotas internas e assina a identidade de vendedor em operações específicas. Configuração e autenticação estão em `backend/fastapi_app/auth/wordpress_bridge.py:10-35`; composição da API em `backend/fastapi_app/main.py:22-45`.

## WooCommerce

WooCommerce é a plataforma de checkout/pedidos no frontend. O plugin sincroniza pedidos pagos em hooks WooCommerce e persiste referências EasyPost; o backend fornece sincronização de pedidos e recebe webhook de pedido. (`frontend/includes/checkout/grpms-checkout.php:1877-1905`; `backend/fastapi_app/routers/checkout/order_router.py:11-25`; `backend/fastapi_app/routers/webhooks/woocommerce_order.py:37-72`.) A assinatura e a idempotência do webhook são responsabilidade do serviço backend `services/checkout/wc_webhook.py`; detalhes em WEBHOOKS.md.

## WCFM

WCFM fornece a função/identidade de vendedor no WordPress (role `wcfm_vendor`); o frontend integra páginas e controles do marketplace com os hooks WCFM. A API usa IDs de vendedor assinados para limitar consultas de rastreio e propriedade. (`frontend/grpms-main.php:907,1266-1320`; `backend/fastapi_app/auth/wordpress_bridge.py:18-35`; `backend/fastapi_app/routers/checkout/easypost_router.py:88-138`.) WCFM não é um serviço HTTP externo chamado diretamente pelo backend, segundo os pontos verificados.

## EasyPost

O backend encapsula criação de endereços/remessas, cálculo/validação de seleção e consulta de tracking por meio de `EasyPostService`. As rotas ficam sob `/api/easypost`; todas exigem segredo WordPress, e endpoints de rastreamento/histórico também verificam identidade/propriedade do vendedor. (`backend/fastapi_app/routers/checkout/easypost_router.py:16-18,53-187`.) Compra de etiqueta está desabilitada e webhook EasyPost retorna 503 (`easypost_router.py:83-86,190-192`). Credenciais/configuração de runtime não são reproduzidas.

## Stripe e pagamentos

Não foi encontrada rota Stripe registrada no app FastAPI. O checkout observado no frontend é baseado em WooCommerce e possui lógica explícita para gateways habilitados e validação de moeda; não é possível atribuir o processamento a Stripe sem configuração de deployment/plugin. (`backend/fastapi_app/main.py:22-45`; `frontend/includes/checkout/grpms-checkout.php:1136-1199`.) Integração Stripe ativa: UNVERIFIED.

## Outras integrações

O backend inclui autenticação JWT, conexão com banco e funções de envio; serviços adicionais podem existir no WordPress. Este documento cobre as integrações solicitadas e o limite evidenciado pelos arquivos citados. Configurações do ambiente ativo: UNVERIFIED.
