Propósito: apresentar a arquitetura e as convenções da API Tuzzello verificadas no código.

Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nesta data.

# Visão geral

A API HTTP é uma aplicação FastAPI em `backend/fastapi_app/main.py`. O módulo monta os roteadores de pedidos WooCommerce, tokens, dashboards, catálogo, checkout, frete, EasyPost e funções móveis. O WordPress/plugin PHP atua como cliente e ponte de identidade; WooCommerce mantém o fluxo de pedidos e checkout. (`backend/fastapi_app/main.py:1-45`; `frontend/includes/checkout/grpms-checkout.php:1877-1905`.)

## Convenções observadas

- Prefixos são definidos por roteador e composição em `main.py`; nem toda rota é versionada no caminho. Não há prefixo global `/v1` configurado. (`backend/fastapi_app/main.py:22-45`.)
- As operações protegidas pelo segredo WordPress exigem `X-GRPMS-WP-Secret`; certos endpoints EasyPost que consultam dados de vendedor também validam a identidade assinada do vendedor. (`backend/fastapi_app/auth/wordpress_bridge.py:10-35`; `backend/fastapi_app/routers/checkout/easypost_router.py:16,88-138`.)
- Erros HTTP usam `detail` do FastAPI; há mensagens específicas documentadas no contrato e nos handlers. Validação Pydantic pode produzir 422. Casos de armazenamento e integração têm respostas explícitas 4xx/5xx nos handlers. (`backend/fastapi_app/routers/token_router.py:13-39`; `backend/fastapi_app/routers/checkout/order_router.py:11-25`.)
- CORS usa `CORS_ALLOWED_ORIGINS`, com origem padrão definida no código; métodos e cabeçalhos são permitidos pelo middleware. (`backend/fastapi_app/main.py:26-33`.)

## Grupos de endpoints

| Área | Rotas registradas | Fonte |
|---|---|---|
| Tokens | `POST /api/token`, `/api/token/login`, `/api/token/from-wp` | `backend/fastapi_app/routers/token_router.py:13-19` |
| Dashboards | `GET /api/dashboard-main/statistics`, `/api/dashboard-main/dashboard/data`, `/api/seller-dashboard/data`, `/api/affiliate-dashboard/data` | `backend/fastapi_app/main.py:35-37`; `backend/fastapi_app/routers/dashboard_main.py:25,33`; routers correspondentes |
| Catálogo | `GET /api/product-gallery`, `GET /api/product-search` | `backend/fastapi_app/routers/product_gallery.py:9-18`; `product_gallery_search.py:9-16` |
| Checkout | `GET /api/checkout/validate-affiliate`; `POST /api/checkout/save-order`, `/api/checkout/save-addresses` | routers `checkout_router.py:15-43`, `order_router.py:8-25` |
| Frete | `GET /api/shipping/get-cost`; endpoints `/api/easypost/...` | `shipping_router.py:9-25`; `easypost_router.py:16,53-192` |
| Mobile | `GET /api/mobile/sales-history`, `GET /api/mobile/profile` | `mobile/mobile_salehistory_router.py:7-10`; `mobile/mobile_profile_router.py:7-10` |
| Webhook | `POST /webhooks/woocommerce/order` | `backend/fastapi_app/routers/webhooks/woocommerce_order.py:37-82` |

## Limites de evidência

As respostas de negócio completas de todos os endpoints não foram inferidas além dos modelos e handlers presentes no código. A especificação formal lista as rotas descobertas estaticamente; detalhes de runtime que dependam de configuração são UNVERIFIED. O schema `/openapi.json` não foi exportado porque não foi comprovado que a aplicação estivesse em execução; também não foi iniciada nenhuma aplicação.

## Relatório de execução

O pedido para Claude Code via `maestri ask` falhou com `Operation not permitted`; `maestri debug` confirmou falha de conexão ao socket Maestri. A lista Maestri identificava Claude Code como agente conectado. Como alternativa, a documentação foi derivada diretamente dos dois HEADs locais: frontend `c1bd75daff802b7ac97bfa32b47f45dd4608d574` e backend `e53fb27c2d11e137eafba8096617a20105320026`. O branch ativo é `main` nos dois repositórios; o backend já continha arquivos não rastreados antes do trabalho, preservados sem alteração.
