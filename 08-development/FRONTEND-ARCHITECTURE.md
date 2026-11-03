# Arquitetura do frontend

**Propósito:** mapear a arquitetura implementada no checkout frontend, que contém um plugin PHP para WordPress e ativos web por módulo.

**Data:** 2026-11-03. Este documento descreve o HEAD de ambos os repositórios nessa data: frontend `c1bd75daff802b7ac97bfa32b47f45dd4608d574` e backend `e53fb27c2d11e137eafba8096617a20105320026`.

## Componentes

1. **Bootstrap do plugin:** `grpms-main.php` declara o plugin, verifica `ABSPATH`, registra hooks e inicializa recursos (frontend: `grpms-main.php:1-12,30-32,1540-1588`).
2. **Módulos PHP:** `includes/` organiza domínios como perfil, vendedor, produtos, checkout, financeiro, galeria, treinamento e dashboard. O módulo de perfil inclui submódulos postal e KYC (frontend: `includes/profile/grpms-profile.php:1-8`).
3. **Templates e ativos:** `templates/` contém templates PHP; CSS e JavaScript são guardados junto dos módulos, como `includes/profile/profile.js` e `includes/profile/profile.css`. Em treinamento, o código registra CSS/JS por página e usa `filemtime` para versões locais (frontend: `includes/training/grpms-training.php:129-163`).
4. **Integração de serviço:** o backend separado é FastAPI. O router de checkout aplica dependência WordPress e delega persistência a `services/checkout/order_service.py` (backend: `fastapi_app/routers/checkout/order_router.py:1-20`).

## Fluxo de execução observado

WordPress carrega o arquivo principal do plugin; os hooks chamam callbacks ao longo do ciclo da plataforma. O autoloader procura classes em diretórios do plugin; no hook `plugins_loaded` são criados alguns serviços e controles de sessão/acesso (frontend: `grpms-main.php:1540-1588`). O conteúdo pode ser conectado a shortcodes, hooks de template e ações AJAX; exemplo de shortcode de perfil (frontend: `includes/profile/grpms-profile.php:69-92`) e registro de rotas AJAX de treinamento (frontend: `includes/training/grpms-training.php:331-358`).

## Limites conhecidos

- A denominação “frontend” deste checkout não significa aplicação React/Vite: a árvore observada é predominantemente PHP, JavaScript, CSS e templates WordPress.
- Não foram encontrados manifests Composer/Node ou documentação arquitetural de build no checkout; compilação de ativos, empacotamento, layout oficial de dependências e suporte de versões são **UNVERIFIED**.
- As interações completas entre cada módulo do plugin e serviços externos não foram inventariadas nesta visão geral. Consulte o router/handler e serviço específico antes de descrever payloads ou efeitos colaterais.
