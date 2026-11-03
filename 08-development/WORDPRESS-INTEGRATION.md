# Integração WordPress, WooCommerce e WCFM

**Propósito:** descrever a integração implementada entre o plugin GRP-MultiSeller, WordPress, WooCommerce, WCFM e o serviço FastAPI.

**Data:** 2026-11-03. Este documento descreve o HEAD de ambos os repositórios nessa data: frontend `c1bd75daff802b7ac97bfa32b47f45dd4608d574` e backend `e53fb27c2d11e137eafba8096617a20105320026`.

## Plugin e ciclo WordPress

O plugin declara cabeçalho `GRP-MultiSeller`, bloqueia execução direta sem `ABSPATH` e registra callbacks nos hooks WordPress (frontend: `grpms-main.php:1-12`). Entre as ações implementadas estão criação de papel de aluno em `init`/ativação, controle de login móvel e redirecionamento de páginas (frontend: `grpms-main.php:14-32,79-84,113-127`). Um autoloader procura classes nos diretórios do plugin e a inicialização ocorre em `plugins_loaded` (frontend: `grpms-main.php:1540-1588`).

Os módulos ficam sob `includes/`, por domínio. Exemplos: perfil registra `[custom_profile]` e restringe conteúdo a usuário conectado e papéis de vendedor/afiliado (frontend: `includes/profile/grpms-profile.php:69-92`); treinamento registra handlers AJAX sob `wp_ajax_*` (frontend: `includes/training/grpms-training.php:331-358`). Cada handler deve ser entendido pelo código do callback correspondente; o registro isolado não prova todas as suas regras de autorização.

## Chamadas ao backend

O aplicativo FastAPI registra routers em `fastapi_app/main.py`. Entre os caminhos explicitamente compostos estão os prefixos `/api/dashboard-main`, `/api/seller-dashboard`, `/api/affiliate-dashboard` e `/api`; outros routers declaram seus próprios prefixos e caminhos (backend: `fastapi_app/main.py:34-48`; `fastapi_app/routers/mobile/mobile_profile_router.py:7-12`).

Exemplo verificável de integração: `POST /api/checkout/save-order` recebe `OrderPayload`, depende de `require_wordpress` no router e delega ao serviço de pedidos; em erro de reconciliação responde HTTP 409 (backend: `fastapi_app/routers/checkout/order_router.py:1-20`). O webhook de pedido WooCommerce é `POST /webhooks/woocommerce/order`, com verificação do evento e limite de corpo definido no código (backend: `fastapi_app/routers/webhooks/woocommerce_order.py:9-10,37-68`).

O middleware CORS lê `CORS_ALLOWED_ORIGINS`; quando ausente, o código define uma origem padrão (backend: `fastapi_app/main.py:26-32`). Não documente chaves ou segredos: nomes de configurações podem ser descritos, seus valores não.

## Dependências e referências

- A documentação oficial WordPress cobre [hooks](https://developer.wordpress.org/plugins/hooks/) e [segurança de plugins](https://developer.wordpress.org/plugins/). A documentação WooCommerce para desenvolvedores está em [developer.woocommerce.com/docs](https://developer.woocommerce.com/docs).
- A documentação do fornecedor WCFM está em [docs.wclovers.com](https://docs.wclovers.com/). Não foi possível confirmar nessa documentação pública um contrato de API específico que corresponda a todos os hooks usados pelo plugin: contratos específicos não verificados são **UNVERIFIED**.
- Requisitos atuais de versão e dependência listados em `README-FE.md` não são validados por manifesto automatizado (frontend: `README-FE.md:11-25`).
