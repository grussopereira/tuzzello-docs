# Padrões de código

**Propósito:** registrar padrões diretamente observáveis no código e nas dependências dos repositórios.

**Data:** 2026-11-03. Este documento descreve o HEAD de ambos os repositórios nessa data: frontend `c1bd75daff802b7ac97bfa32b47f45dd4608d574` e backend `e53fb27c2d11e137eafba8096617a20105320026`.

## PHP e WordPress

- Os arquivos PHP usam guardas de acesso direto com `ABSPATH` em módulos como `grpms-main.php` e `includes/profile/grpms-profile.php` (frontend: `grpms-main.php:9-12`; `includes/profile/grpms-profile.php:1-5`).
- O plugin conecta callbacks aos hooks WordPress por `add_action` e `add_filter`; módulos também registram shortcodes e ações AJAX (frontend: `grpms-main.php:30-32`; `includes/profile/grpms-profile.php:69-71`; `includes/training/grpms-training.php:331-358`).
- Trechos existentes validam autorização antes de fornecer perfil e escapam valores ao compor HTML/e-mails (frontend: `includes/profile/grpms-profile.php:73-92`; `includes/seller/grpms-seller.php:49-82`). Para novos fluxos, escolha funções WordPress de validação, sanitização e escape compatíveis com o tipo/contexto. A orientação oficial é [Segurança no WordPress](https://developer.wordpress.org/apis/security/).
- O estilo não é uniforme em todo o plugin. Não foi localizada regra versionada de PHPCS, formatter ou configuração PHP equivalente; regras automáticas e estilo obrigatório: **UNVERIFIED**.

## Python e FastAPI

- As dependências de runtime têm versões exatas em `requirements.txt`; dependências de desenvolvimento são acrescentadas por `requirements-dev.txt` (backend: `requirements.txt:1-14`; `requirements-dev.txt:1-3`).
- Os routers usam `APIRouter`, modelos Pydantic e `Depends` para validação e injeção de autenticação; operações de domínio aparecem em módulos de serviço separados (backend: `fastapi_app/routers/checkout/order_router.py:1-25`; `fastapi_app/routers/mobile/mobile_profile_router.py:1-23`).
- A configuração do banco seleciona MySQL (padrão) ou PostgreSQL via `DB_ENGINE`, e lê parâmetros do ambiente sem valores versionados no código (backend: `fastapi_app/utils/database.py:8-30`). Não copie valores de `.env` para documentação ou logs.
- Nenhuma configuração versionada de formatter, linter ou type checker foi confirmada na inspeção: **UNVERIFIED**.

## JavaScript, CSS e ferramentas

- Os ativos do plugin estão organizados junto aos módulos em `includes/<domínio>/js`, `css` e, em algumas áreas, `vendor` (por exemplo, `includes/profile/profile.js:1-18`, `includes/profile/profile.css:1-15` e o Select2 distribuído em `includes/products/vendor/select2.min.js:1`).
- Não foi encontrado `package.json` no frontend analisado; scripts npm e versões padronizadas de Node: **UNVERIFIED**.
- Não imponha neste documento convenções de indentação, nomes ou automação não confirmadas em arquivos de configuração do repositório.
