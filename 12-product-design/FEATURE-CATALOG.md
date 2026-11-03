# Catálogo de funcionalidades

Propósito: listar módulos e interfaces visíveis no HEAD dos repositórios examinados.

> Documento de referência do HEAD dos repositórios frontend e backend em 2026-11-03.

| Área | Implementação observável | Evidência |
|---|---|---|
| Dashboard e métricas | Shortcodes para gráficos de vendas, top produtos e ganhos de afiliado; ações AJAX para dados/estatísticas. | `frontend/includes/dashboard/grpms-dashboard.php:12-76` |
| Escopo do dashboard | Resolução de contexto de usuário e escopo para vendor, afiliado e gerente da loja. | `frontend/includes/dashboard/grpms-dashboard.php:78-100` |
| Dashboard seller | Arquivo agregador inclui configuração de frete, pedidos, loja e dashboard seller. | `frontend/includes/seller/grpms-seller.php:7-12` |
| Checkout | Código frontend identifica produto e vendedor; backend inclui routers checkout e endpoints de persistência de pedido/endereço. | `frontend/includes/checkout/grpms-checkout.php:52-74`; `backend/fastapi_app/routers/checkout/order_router.py:8-25` |
| Galeria e busca | Routers de galeria e pesquisa incluídos na aplicação FastAPI. | `backend/fastapi_app/main.py:40-42` |
| Frete e EasyPost | Routers de shipping e EasyPost registrados no app. | `backend/fastapi_app/main.py:43-46` |
| Experiência mobile | Routers de perfil e histórico de vendas mobile registrados. | `backend/fastapi_app/main.py:47-48` |
| Webhook WooCommerce | Router de webhook de pedido incluído no app. | `backend/fastapi_app/main.py:21-24` |
| Treinamento de produto | Há templates de currículo, preview de treinamento e classes de treinamento no frontend. | `frontend/templates/product-training-preview.php`; `frontend/templates/product-training-curriculum.php`; `frontend/includes/training/grpms-training.php` |

Detalhes funcionais não dedutíveis dos arquivos citados, disponibilidade em produção e dependências configuradas: **UNVERIFIED**.
