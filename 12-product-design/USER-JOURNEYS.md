# Jornadas de usuário

Propósito: descrever os fluxos de interface e API identificáveis no código, sem presumir comportamento externo.

> Documento de referência do HEAD dos repositórios frontend e backend em 2026-11-03.

## Vendedor ou afiliado consulta indicadores

1. A página pode incluir os shortcodes de gráfico de vendas, produtos de destaque ou ganhos de afiliado. (`frontend/includes/dashboard/grpms-dashboard.php:12-49`)
2. O frontend resolve autorização para estatísticas e, se autorizado, carrega JS/CSS e publica URL AJAX, nonce, papel e moeda no script. (`frontend/includes/dashboard/grpms-dashboard.php:51-73`)
3. As ações AJAX registradas encaminham solicitações de dados e estatísticas. (`frontend/includes/dashboard/grpms-dashboard.php:75-76`)
4. A resolução de escopo diferencia vendor, affiliate e shop manager com capacidade de gerenciamento WooCommerce. (`frontend/includes/dashboard/grpms-dashboard.php:86-100`)

A sequência completa da tela e respostas exibidas depende do JavaScript e de serviços não detalhados aqui: **UNVERIFIED**.

## Checkout e gravação do pedido

1. O frontend tem código de checkout que identifica produto atual e vendedor associado em contexto de carrinho/sessão ou usuário. (`frontend/includes/checkout/grpms-checkout.php:52-74`)
2. O backend expõe `POST /api/checkout/save-order`, protegido por `require_wordpress`. (`backend/fastapi_app/routers/checkout/order_router.py:8-12`)
3. O serviço de reconciliação grava dados; falhas de reconciliação são convertidas em HTTP 409 e outras exceções em HTTP 500. (`backend/fastapi_app/routers/checkout/order_router.py:13-20`)
4. O endpoint `POST /api/checkout/save-addresses` passa endereço de cobrança e entrega ao serviço de persistência. (`backend/fastapi_app/routers/checkout/order_router.py:23-25`)

A experiência visível ao comprador e a ordem exata das chamadas no navegador: **UNVERIFIED**. Nenhuma chamada a provedor externo foi realizada.

## Usuário consulta catálogo

A aplicação inclui routers de galeria e busca de produtos. (`backend/fastapi_app/main.py:40-42`) Os parâmetros e passos da jornada são **UNVERIFIED** neste resumo.
