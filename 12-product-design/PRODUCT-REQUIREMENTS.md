# Requisitos do produto

Propósito: registrar capacidades e restrições de produto que podem ser confirmadas no código atual.

> Documento de referência do HEAD dos repositórios frontend e backend em 2026-11-03.

## Escopo confirmado

- O plugin frontend integra funcionalidades de marketplace construídas sobre WordPress/WooCommerce; o dashboard declara atender vendedores e afiliados. (`frontend/includes/dashboard/grpms-dashboard.php:1-6`)
- Os gráficos de vendas, produtos mais vendidos e ganhos de afiliados são expostos por shortcodes distintos, sujeitos a validação de acesso. (`frontend/includes/dashboard/grpms-dashboard.php:12-49`)
- O backend FastAPI registra routers de dashboards, checkout, catálogo, frete, autenticação/token, mobile e webhook de pedido. (`backend/fastapi_app/main.py:9-48`)
- O salvamento de pedidos pelo endpoint `/api/checkout/save-order` requer a dependência `require_wordpress`; retorna 204 em sucesso, 409 para reconciliação necessária e 500 para falha não classificada. (`backend/fastapi_app/routers/checkout/order_router.py:8-20`)

## Requisitos observáveis

1. Acesso às estatísticas do dashboard deve ser autorizado por escopo e capacidade; o código determina escopo de vendedor, afiliado ou gerência. (`frontend/includes/dashboard/grpms-dashboard.php:86-100`)
2. Os dados dos gráficos devem ser carregados por scripts vinculados ao usuário autorizado e às rotas AJAX registradas. (`frontend/includes/dashboard/grpms-dashboard.php:51-76`)
3. O frontend comunica operações de checkout e domínios correlatos ao backend via routers incluídos na aplicação. (`backend/fastapi_app/main.py:34-48`)
4. A API recebe origens CORS de `CORS_ALLOWED_ORIGINS`, usando uma origem padrão definida no código quando a variável não está presente. (`backend/fastapi_app/main.py:26-32`)

## Fora do que pode ser afirmado

Regras comerciais completas, SLAs, métricas-alvo, critérios de aceite de negócio, planos pagos e obrigações legais: **UNVERIFIED**. Este documento descreve comportamento encontrado no código, não requisitos aprovados por negócio.
