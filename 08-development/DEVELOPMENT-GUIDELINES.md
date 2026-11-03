# Diretrizes de desenvolvimento

**Propósito:** orientar a leitura e a contribuição técnica nos repositórios frontend (plugin WordPress) e backend (serviço FastAPI).

**Data:** 2026-11-03. Este documento descreve o HEAD de ambos os repositórios nessa data: frontend `c1bd75daff802b7ac97bfa32b47f45dd4608d574` e backend `e53fb27c2d11e137eafba8096617a20105320026`.

## Estrutura observada

- O frontend é o plugin WordPress `GRP-MultiSeller`, com o ponto de entrada `grpms-main.php` e módulos agrupados por domínio em `includes/` (frontend: `grpms-main.php:1-12`; exemplos: `includes/profile/grpms-profile.php:1-8`, `includes/seller/grpms-seller.php:1-12`). O ponto de entrada registra hooks, carrega módulos e implementa comportamentos de acesso e navegação (frontend: `grpms-main.php:30-32`, `grpms-main.php:1563-1588`).
- O backend é um aplicativo FastAPI: `fastapi_app/main.py` instancia a aplicação e registra routers de dashboard, checkout, galeria, webhooks e mobile (backend: `fastapi_app/main.py:23-48`). Routers delegam operações aos serviços do domínio, como sincronização de pedidos (backend: `fastapi_app/routers/checkout/order_router.py:1-25`).
- Os arquivos `README-FE.md` e `README-BE.md` no frontend são notas de referência existentes. Seus requisitos e exemplos não são tratados como configuração atual sem confirmação no código e nas dependências (frontend: `README-FE.md:11-25`; `README-BE.md:1-20`).

## Como contribuir com evidência

1. Identifique o módulo responsável antes de alterar comportamento: no WordPress, siga o hook/shortcode/AJAX até o módulo; no FastAPI, siga o router até serviço e persistência.
2. Preserve contratos observados. Rotas são registradas centralmente em `main.py`; dependências de autenticação são declaradas nos routers ou funções de dependência (backend: `fastapi_app/main.py:34-48`; `fastapi_app/routers/checkout/order_router.py:8-9`).
3. Ao tratar dados de entrada, autorização, sessão, operações financeiras, banco, arquivos ou integrações, consulte os controles já usados no fluxo afetado. O código PHP combina validação de acesso e sanitização/escape conforme o contexto (frontend: `includes/profile/grpms-profile.php:69-92`; `includes/seller/grpms-seller.php:14-28`).
4. Registre no material de mudança o caminho e as linhas da implementação que sustentam cada afirmação. Diferencie comportamento implementado, hipótese e proposta.

## Verificações de desenvolvimento

- O backend mantém dependências fixadas em `requirements.txt`; `requirements-dev.txt` inclui pytest e httpx (backend: `requirements.txt:1-14`; `requirements-dev.txt:1-3`). O repositório contém testes em `tests/`, mas este documento não afirma que uma suíte foi executada.
- Não foi encontrada configuração versionada de lint/formatação, teste PHP ou dependências Node/Composer no nível inspecionado do frontend. Ferramentas e comandos padronizados de validação do plugin: **UNVERIFIED**.
- Procedimentos de instalação, provisionamento, deploy e infraestrutura que não estejam descritos nos arquivos de código/documentação citados: **UNVERIFIED**.

## Referências oficiais

- [WordPress Plugin Developer Handbook](https://developer.wordpress.org/plugins/): hooks e tópicos de segurança.
- [WooCommerce Developer Documentation](https://developer.woocommerce.com/docs): referência oficial para extensão WooCommerce.
- [WCFM Documentation](https://docs.wclovers.com/): documentação oficial do produto WCFM.
