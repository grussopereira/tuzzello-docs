> **STATUS DO DOCUMENTO: PARCIAL** (revisado em 2026-11-03 após validação ARCH-VALIDATE-01 / ARCH-FIX-01). Citações de linha corrigidas contra o HEAD de frontend e backend; o modelo de implantação descrito é o evidenciado pelas docs de implantação (VMs, não contêineres). Itens não confirmáveis no código estão marcados **NAO VERIFICADO**: Nginx, bloqueio público de rotas, status de produção de qualquer componente, Stripe/WooPayments, PayPal, Podman, YITH Affiliates, topologia/IPs, cron (apenas em README-FE.md), instalação do MU-Plugin de mídia. Flags com padrão desligado (`GRPMS_COMMISSION_WATERFALL`, `EASYPOST_BUY_LIVE`, `GRPMS_AUTO_COMPLETE`, `GRPMS_WC_SEND_ENABLED`) estão descritas como tal.
> **ARCH-FIX-04 (2026-11-03) — Stripe e systemd:** (a) Stripe (ARCH-FIX-05): segundo o Product Owner (informação do proprietário, aceita como declarada), o Stripe é o provedor de pagamento da plataforma, usado por meio de um plugin do WooCommerce. O mecanismo dentro do WordPress (qual plugin, versão e configuração) é **NAO VERIFICADO**: não está nestes repositórios. Nenhuma chamada à API Stripe, PaymentIntent, Checkout Session, webhook Stripe ou verificação de assinatura existe em `frontend/` e `backend/fastapi_app` neste HEAD (apenas fixtures em frontend/tests/payout-approval-guard.php:367,395); o checkout guarda o identificador genérico de método de pagamento do WooCommerce e o backend guarda o `transaction_id` recebido do webhook de pedido do WooCommerce. O ponto de integração é o WooCommerce, não o backend. Referências de configuração fora do código (somente nomes de arquivo e linhas; nenhum conteúdo aberto ou registrado): backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (drop-in `stripe.conf`) e :24 (arquivo `backend/infrastructure/secrets/stripe.env`, referência de arquivo apenas, ausente neste workspace); frontend/docs/REGISTRO-ERROS-CORRECOES.md:579. (b) systemd: não consta em CLOUD-DEPLOYMENT.md (frontend e backend); consta em backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (alegação do repositório lab, conflitante com a decisão do PO `grpms-backend`: serviço de usuário `tuzzello-backend`, **NAO VERIFICADO**) e frontend/docs/REGISTRO-ERROS-CORRECOES.md:579; nome de serviço: decisão do PO = `grpms-backend`; a citação `tuzzello-backend` (PROCESSO-AUTOMATICO-CONCLUSAO.md:22) é alegação do repositório conflitante, NAO VERIFICADO; código/implantação: **NAO VERIFICADO**. (c) Citações de arquivo usam caminho completo a partir da raiz do repositório (`frontend/…`, `backend/…`).
> **ARCH-NOTE (2026-11-03) — Nome do serviço (conflito resolvido):** o Product Owner decidiu `grpms-backend` (2026-11-03, decisão vinculante). O serviço de usuário systemd do LAB foi renomeado de `tuzzello-backend` para `grpms-backend` em 2026-11-03 (verificado somente leitura pelo relatório do engenheiro `/home/gilberto/tuzzello-specs/eng-rename-report.md`). A linha citada do documento do backend (`backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22`) foi atualizada no commit e75597f do repositório backend. Demais afirmações e o status **PARCIAL** do documento permanecem inalterados.

> **ARCH-FIX-03 (2026-11-03):** releitura afirmação por afirmação contra `frontend/` e `backend/`. Convenção: **VERIFICADO (arquivo:linha)** = confirmado no código; **NAO VERIFICADO** = não confirmado por código. **Modelo de implantação (VMs `lptaf01web01`/`lptaf01pyt01`, serviço `grpms-backend`): NAO VERIFICADO no código; fonte: documentação de implantação e operações** (frontend docs/CLOUD-DEPLOYMENT.md:3-4; backend docs/CLOUD-DEPLOYMENT.md:3, :54, :59). **`systemd` NÃO consta em backend/docs/CLOUD-DEPLOYMENT.md nem em frontend/docs/CLOUD-DEPLOYMENT.md; consta em backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (alegação do repositório lab, conflitante com a decisão do PO `grpms-backend`: serviço de usuário `tuzzello-backend`, **NAO VERIFICADO**) e é citado em frontend/docs/REGISTRO-ERROS-CORRECOES.md:579 (EnvironmentFile do host). Discrepância de nome de serviço: `grpms-backend` (CLOUD-DEPLOYMENT.md:54, :59) vs `tuzzello-backend` (PROCESSO-AUTOMATICO-CONCLUSAO.md:22, alegação conflitante, NAO VERIFICADO); decisão do PO: `grpms-backend`. Uso de systemd no código/implantação: NAO VERIFICADO**. Restam afirmações NAO VERIFICADAS (Stripe/PayPal, implantação, produção, cron, MU-Plugin), portanto o status permanece **PARCIAL**.

# Visão Geral da Arquitetura do Sistema Tuzzello

> **Propósito:** Documentar a arquitetura de software do ecossistema Tuzzello (Marketplace MultiSeller). **Data:** 2026-11-03. Descreve o código no HEAD dos repositórios Frontend (/home/gilberto/Projects/Tuzzello-AI-Lab/frontend) e Backend (/home/gilberto/Projects/Tuzzello-AI-Lab/backend).

---

## 1. Contexto do Sistema

O Tuzzello é um **marketplace multi-vendedor** construído sobre WordPress + WooCommerce + WCFM (WC Frontend Manager), com um backend de serviços em FastAPI (Python) para operações especializadas que não podem ser executadas diretamente no WordPress por questões de segurança, desempenho ou complexidade.

**Domínio:** E-commerce B2C multi-vendedor (descrição de domínio do arquiteto, NAO VERIFICADA como "B2C"); suporte a (marcadores por item abaixo):
- Vendedores individuais e lojas (Store Sellers): VERIFICADO `grpms_get_seller_account_type` (frontend/grpms-main.php:935)
- Afiliados (YITH Affiliates): VERIFICADO apenas a role `yith_affiliate` (grpms-affiliation.php:17, checkout_service.py:24); plugin YITH NAO VERIFICADO
- Produtos físicos e digitais: VERIFICADO em parte (produtos digitais/virtuais forçados para vendedores, frontend/grpms-main.php:1264; frete físico em easypost_service.py:353; ver 5.1)
- Multi-moeda (EUR, USD, BRL, ZAR, XOF): VERIFICADO frontend/grpms-main.php:808
- Frete internacional (EasyPost): VERIFICADO em parte (cotação easypost_router.py:141; compra de etiqueta desativada, :83-85)
- Comissões em cascata (VERIFICADO backend/fastapi_app/services/checkout/commission_math.py:8-11: plataforma, depois afiliado sobre o restante, vendedor com o resto; desligadas por padrão, backend/fastapi_app/services/checkout/order_service.py:133-140; a ordem "vendor → plataforma → afiliado" anterior estava incorreta)

**Página inicial do plugin:** `frontend/grpms-main.php:1-7` (VERIFICADO; a descrição do cabeçalho do plugin é "Extensões para WCFM...", :3-4 região)

**Nota de unidade de implantação:** onde este documento dizia "container", o código e a documentação de implantação evidenciam VMs (ver seção 3).

---

## 2. Visão Geral dos Componentes

### 2.1. Frontend (WordPress Plugin: GRP-MultiSeller)

| Camada | Tecnologia | Responsabilidade | Localização |
|--------|------------|------------------|-------------|
| Apresentação | PHP + HTML/CSS/JS (VERIFICADO: diretórios `templates/`, `css/`, `js/`, `html/` existem) | Templates, pages, assets | `templates/`, `css/`, `js/` |
| Lógica de Negócio | PHP (versão em runtime NAO VERIFICADA; "8.x" é da documentação) | Hooks WordPress, lógica de marketplace (VERIFICADO: `includes/*`, ver COMPONENT-CATALOG.md) | `includes/*` |
| Integração | PHP + HTTP | Clients para backend FastAPI | `includes/api/grpms-backend-client.php` |
| Dados | MySQL/MariaDB (wp_*) — motor NAO VERIFICADO no código (o código mostra apenas pymysql; o papel do projeto cita MariaDB) | Tabelas WordPress + WCFM + Custom | via `$wpdb` |

**Plugin principal:** `frontend/grpms-main.php` (VERIFICADO Versão 1.1.0, linha 5; citação anterior "linha 3" corrigida)
**Requisitos:** WordPress 6.0+, WooCommerce 7.0+ (README-FE.md:13-14; documentação, NAO VERIFICADO no código). WCFM não consta da lista de requisitos de README-FE.md:13-26 (NAO VERIFICADO como requisito declarado)

### 2.2. Backend (FastAPI Service)

| Camada | Tecnologia | Responsabilidade | Localização |
|--------|------------|------------------|-------------|
| API | FastAPI + Python (testes locais em 3.11; VM usa 3.13 — backend docs/CLOUD-DEPLOYMENT.md:75-76) | Endpoints REST | `fastapi_app/main.py` |
| Serviços | Python | Lógica de negócios complexa | `fastapi_app/services/*` |
| Autenticação | JWT RS256 | Geração de tokens | `fastapi_app/auth/jwt_rs256.py` |
| Dados | MySQL (pymysql); caminho PostgreSQL (psycopg2) existe no código mas o SQL é específico de WordPress/MySQL (tabelas `wp_*`, ex.: token_router.py:24) | Acesso direto ao BD WordPress | `fastapi_app/utils/database.py` |

**Framework:** FastAPI com Pydantic (VERIFICADO main.py:2, models.py, requirements.txt:1,:3)
**Banco de dados:** `get_connection()` seleciona pymysql (padrão, `DB_ENGINE=mysql`) ou psycopg2 (`DB_ENGINE=postgresql`) (database.py:8-30). O esquema consultado é o do WordPress/MySQL; suporte efetivo a PostgreSQL NAO VERIFICADO.
**Autenticação com WordPress:** VERIFICADO `fastapi_app/auth/wordpress_bridge.py:10-15` (`require_wordpress`, verificação via `X-GRPMS-WP-Secret`; comparação em :14). Existe também um segundo mecanismo, de identidade de vendedor (ver seção 6.1).

---

## 3. Topologia de Implantação

```
┌─────────────────────────────────────────────────────────────────┐
│                      Navegador do Usuário                          │
└─────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│        WordPress (novo.tuzzello.com; VM lptaf01web01)             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐   │
│  │   WooCommerce   │  │     WCFM        │  │   GRP-MultiSeller│  │
│  │   (Núcleo)      │  │   (Multi-vendedor)│  │    (Plugin)     │   │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘   │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    MySQL (tuzzello_db)                       │  │
│  └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
            ┌───────────┐   ┌───────────┐   ┌───────────┐
            │ Stripe via│   │ PayPal    │   │ EasyPost  │
            │ WooComm.NV│   │ (Pagamento)│   │  (Frete)  │
            └───────────┘   └───────────┘   └───────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│   Backend FastAPI (nf-novo.tuzzello.com; VM lptaf01pyt01)        │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐   │
│  │  Routers        │  │   Services      │  │   Database      │   │
│  │  (API REST)     │  │   (Lógica)      │  │   (MySQL)       │   │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

> **Marcadores do diagrama (ARCH-FIX-03):** hostnames `novo.tuzzello.com`/`nf-novo.tuzzello.com`, VMs e nome de banco `tuzzello_db` (backend docs/CLOUD-DEPLOYMENT.md:32): documentação, NAO VERIFICADO no código (o domínio `novo.` aparece só como valor padrão de CORS, main.py:28). Caixas WooCommerce, WCFM, MySQL: NAO VERIFICADO (plugins/banco externos ao código). Caixa GRP-MultiSeller e Backend (Routers, Services, Database): VERIFICADO. Setas para Stripe e PayPal: NAO VERIFICADO (nenhuma chamada nos repositórios). Seta para EasyPost: VERIFICADO, mas parte do backend (easypost_service.py:10), não do WordPress como o desenho sugere. "Database (MySQL)" no backend: driver `pymysql` VERIFICADO, MySQL NAO VERIFICADO.

> **Modelo de implantação (conforme a documentação de implantação, não verificável no código):** WordPress e backend rodam em **VMs** separadas (frontend `docs/CLOUD-DEPLOYMENT.md:3-4`: `lptaf01web01`; backend `docs/CLOUD-DEPLOYMENT.md:3`: `lptaf01pyt01`, serviço `grpms-backend` (backend docs/CLOUD-DEPLOYMENT.md:54, :59; "systemd": `systemd` NÃO consta em backend/docs/CLOUD-DEPLOYMENT.md nem em frontend/docs/CLOUD-DEPLOYMENT.md; consta em backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (alegação do repositório lab, conflitante com a decisão do PO `grpms-backend`: serviço de usuário `tuzzello-backend`, **NAO VERIFICADO**) e é citado em frontend/docs/REGISTRO-ERROS-CORRECOES.md:579 (EnvironmentFile do host). Discrepância de nome de serviço: `grpms-backend` (CLOUD-DEPLOYMENT.md:54, :59) vs `tuzzello-backend` (PROCESSO-AUTOMATICO-CONCLUSAO.md:22, alegação conflitante, NAO VERIFICADO); decisão do PO: `grpms-backend`. Uso de systemd no código/implantação: NAO VERIFICADO), venv Python, deploy por rsync). **Não** há evidência de implantação em contêineres: `backend/podman` é um arquivo vazio (0 bytes), não um diretório. Topologia física, endereços de rede e Nginx: NAO VERIFICADO.

**Configuração de rede:**
- URL pública backend: `GRPMS_API_BASE_URL` (ex: `https://nf-novo.tuzzello.com`, valor de exemplo da documentação)
- URL interna backend: `GRPMS_API_INTERNAL_URL` (valor de exemplo existe apenas na documentação de implantação, frontend docs/CLOUD-DEPLOYMENT.md:46; endereço omitido de propósito; NAO VERIFICADO no código)
- Segredo compartilhado: `GRPMS_WP_SHARED_SECRET` (VERIFICADO: 32+ caracteres exigidos em wordpress_bridge.py:12 e grpms-backend-client.php:29)
- CORS: variável `CORS_ALLOWED_ORIGINS` (VERIFICADO backend/.env.example:9 e main.py:26-32; o valor `https://novo.tuzzello.com` é o padrão em main.py:28)

**Integração WordPress ↔ Backend:**
- Frontend → Backend: via `grpms_backend_http()` em `includes/api/grpms-backend-client.php:27-` (rotas permitidas em :33-35; header `X-GRPMS-WP-Secret` adicionado em :46-48)
- Backend → WordPress: acesso direto ao BD via `get_connection()` em `database.py:8-30` (VERIFICADO; não há chamada HTTP do backend ao WordPress, exceto o cliente WooCommerce "LAB-only" backend/fastapi_app/services/checkout/wc_client.py:140, desligado por padrão)
- Autenticação: Header `X-GRPMS-WP-Secret` validado em `wordpress_bridge.py:10-15`

---

## 4. Tecnologias e Versões

### 4.1. Frontend Stack

| Componente | Tecnologia | Versão | Fonte |
|------------|------------|--------|-------|
| CMS | WordPress | 6.0+ | README-FE.md:13 |
| E-commerce | WooCommerce | 7.0+ | README-FE.md:14 |
| Multi-vendedor | WCFM | - | não consta em README-FE.md:13-26 (NAO VERIFICADO) |
| Pagamentos | WooPayments | 9.0+ (documentação; NAO VERIFICADO no código) | README-FE.md:15 |
| Pagamentos | WooCommerce PayPal | 3.1.0+ (documentação; NAO VERIFICADO no código) | README-FE.md:16 |
| Builder | Elementor | 3.0+ | README-FE.md:17 |
| Formulários | WPForms Lite | 1.9+ | README-FE.md:21 |
| Tradução | GTranslate | 3.0+ | README-FE.md:22 |
| 2FA | miniOrange | 6.0+ | README-FE.md:23 |
| JWT Auth | JWT Authentication for WP-API | 1.4+ (documentação; o código usa JWT próprio do backend, token_router.py:35; uso do plugin NAO VERIFICADO) | README-FE.md:24 |
| Gerenciador | Header Footer Code Manager | 1.1.42+ | README-FE.md:25 |
| GDPR | Complianz | 7.4.2+ | README-FE.md:26 |
| PHP | PHP | 8.4 na VM nova (frontend docs/CLOUD-DEPLOYMENT.md:82 "validar PHP 8.4"); o crontab de README-FE.md:249 recarrega `php8.2-fpm` (VM antiga? NAO VERIFICADO) | frontend docs/CLOUD-DEPLOYMENT.md:82; README-FE.md:249 |
| JS | Vanilla JS + jQuery | - (jQuery NAO VERIFICADO; JS em `js/` e `includes/*/js` VERIFICADO) | Arquivos em `js/` |
| CSS | CSS3 + SASS | - (SASS NAO VERIFICADO; CSS em `css/` VERIFICADO) | Arquivos em `css/` |

### 4.2. Backend Stack

| Componente | Tecnologia | Versão | Fonte |
|------------|------------|--------|-------|
| Framework | FastAPI | 0.141.1 (VERIFICADO requirements.txt:1) | backend/requirements.txt |
| ASGI | Uvicorn | 0.54.0 (VERIFICADO requirements.txt:2; uso em runtime NAO VERIFICADO) | backend/requirements.txt |
| Autenticação | JWT RS256 (python-jose 3.5.0, VERIFICADO requirements.txt:5; algoritmo jwt_rs256.py:16) | - | auth/jwt_rs256.py |
| Serialização | Pydantic | 2.13.5 | backend/requirements.txt; models.py |
| Banco | MySQL via PyMySQL | PyMySQL 1.2.3; versão do servidor NAO VERIFICADA | requirements.txt; database.py:19-27 |
| Banco | PostgreSQL via psycopg2-binary | 2.9.13; uso real NAO VERIFICADO | requirements.txt; database.py:11-18 |
| HTTP | httpx | NAO VERIFICADO em requirements.txt (VERIFICADO: não consta); porém é importado em backend/fastapi_app/services/checkout/wc_client.py:19 — dependência declarada ausente | - |
| Serialização PHP | phpserialize | 1.3 | token_router.py:3; requirements.txt |

### 4.3. Infraestrutura

| Serviço | Tecnologia | Observações | Fonte |
|---------|------------|-------------|-------|
| Servidor Web | Nginx | Citado apenas na documentação de implantação do backend (bloqueios públicos de rotas); configuração NAO VERIFICADA | backend docs/CLOUD-DEPLOYMENT.md:33 |
| PHP-FPM | PHP 8.4 (alvo na VM nova) | - | frontend docs/CLOUD-DEPLOYMENT.md:82 |
| Aplicação | VMs (backend: serviço `grpms-backend` segundo CLOUD-DEPLOYMENT.md; systemd NAO VERIFICADO, ver nota em :3; venv, rsync) | Podman NAO VERIFICADO: `backend/podman` é um arquivo vazio (0 bytes), não um diretório | backend docs/CLOUD-DEPLOYMENT.md |
| Cloud | GCP (citado na documentação) | VMs lptaf01web01 (web) e lptaf01pyt01 (backend) | frontend docs/CLOUD-DEPLOYMENT.md:3; backend docs/CLOUD-DEPLOYMENT.md:3 |
| Cron | crontab (não systemd timers) | Tarefas agendadas; apenas descritas em README-FE.md, NAO VERIFICADO | README-FE.md:228-249 |

---

## 5. Limites e Restrições do Sistema

### 5.1. Limites de Negócio

1. **Produtos físicos:** 1 produto/1 unidade por cotação de frete (frontend docs/CLOUD-DEPLOYMENT.md:22; documentação, NAO VERIFICADO no código)
   - UNVERIFIED: Regra de embalagem para múltiplas unidades não implementada

2. **Moedas:** Apenas tarifas na mesma moeda do checkout são aceitas (frontend docs/CLOUD-DEPLOYMENT.md:24; documentação, NAO VERIFICADO no código)
   - Conversão de moeda para frete exige política cambial verificável

3. **Frete internacional:** Bloqueado até uso de dados reais do produto (frontend docs/CLOUD-DEPLOYMENT.md:26; documentação, NAO VERIFICADO no código)
   - Origem completa do vendedor é obrigatória

4. **Produtos virtuais:** Não requerem frete (frontend docs/CLOUD-DEPLOYMENT.md:27; documentação; VERIFICADO em parte: grpms_cart_requires_shipping_server, grpms-checkout-guard.php:47)

5. **Vendedores:** Apenas podem cadastrar produtos virtuais (VERIFICADO frontend/grpms-main.php:1263-1264)
   - `grpms_should_force_vendor_virtual_products()`: vendedores logados (não admins) têm produtos forçados como virtuais

6. **Acesso móvel:** Bloqueado para áreas de gestão (VERIFICADO frontend/grpms-main.php:80-127 e :1357-1362)
   - `/mydash/*` e `/app/*` requerem desktop
   - Redireciona para `/no-mobile-access`

### 5.2. Limites Técnicos

1. **Sessão de cotação:** 15 minutos de validade (frontend docs/CLOUD-DEPLOYMENT.md:12; documentação, NAO VERIFICADO no código)
   - Alteração de carrinho/endereço exige recálculo

2. **Tokens de painel:** Renovados por AJAX com nonce WordPress (VERIFICADO grpms-backend-client.php:93, :110, ação `wp_ajax_grpms_dashboard_token`)
   - Não expõem segredo nem endereço interno (VERIFICADO em parte: o JS recebe `apiUrl` público de `GRPMS_API_BASE_URL`, frontend/includes/seller/grpms-seller-dashboard.php:171-186; o segredo compartilhado fica no servidor, grpms-backend-client.php:27-29)

3. **Sincronização de pedidos:** Após pagamento, não ao abrir página de agradecimento (frontend docs/CLOUD-DEPLOYMENT.md:17; documentação, NAO VERIFICADO no código)

4. **EasyPost:** Modo teste por padrão (VERIFICADO `EASYPOST_MODE=test` no exemplo, backend/.env.example:10; padrão em runtime NAO VERIFICADO)
   - Compra de etiquetas e webhooks EasyPost desativados/503 (easypost_router.py:85, :192; frontend docs/CLOUD-DEPLOYMENT.md:28)

5. **Comissão em cascata:** Desativada por padrão (VERIFICADO `GRPMS_COMMISSION_WATERFALL=0`; comentário em backend/.env.example:14, flag em :15; padrão no código backend/fastapi_app/services/checkout/order_service.py:136)
   - Afeta dinheiro de afiliados e vendedores
   - Comentário da fonte: "needs PO authorisation" (.env.example:14). Definição de "PO": NAO VERIFICADO (o termo também aparece em frontend docs/REGISTRO-ERROS-CORRECOES.md:339 sem definição; nenhum documento desta pasta o define; significado não inventado) (confirmar com o Product Owner)

### 5.3. Segurança

1. **Segredo compartilhado:** Nunca exposto no frontend (VERIFICADO: o segredo é constante PHP lida só no servidor, grpms-backend-client.php:27-29; header adicionado em :46-48; exposição por outros caminhos NAO VERIFICADA)
   - Validado em `wordpress_bridge.py:10-15`

2. **Acesso à mídia:** Limitado ao proprietário — **apenas documentado** (README-FE.md:~102-104 em diante contém instruções e código; NAO VERIFICADO como implementado)
   - A documentação diz que apenas administrator e shop_manager teriam acesso irrestrito
   - O MU-Plugin `wp-content/mu-plugins/grpms-limit-media-to-owner.php` **não existe nos repositórios**; existe só como texto no README. Instalação em qualquer ambiente: NAO VERIFICADO

3. **Tokens JWT:** Expiração configurável (VERIFICADO `JWT_EXPIRE_MINUTES` em .env.example:8; padrão 15 em jwt_rs256.py:17)

4. **CORS:** Origens permitidas configuráveis (VERIFICADO main.py:26-32)

---

## 6. Fluxos de Dados Principais

| Fluxo | Origem | Destino | Protocolo | Autenticação |
|-------|--------|---------|-----------|---------------|
| Requisição de token | WordPress (PHP, servidor) | Backend FastAPI | HTTP POST | `X-GRPMS-WP-Secret` |
| Validação de afiliado (VERIFICADO: HTTP **GET**, grpms-checkout.php:1287, grpms-guest-shipping.php:218) | WordPress (PHP, servidor) | Backend | HTTP GET | `X-GRPMS-WP-Secret` |
| Cálculo/validação de frete (VERIFICADO grpms-guest-shipping.php:107, :141) | WordPress (PHP, servidor) | Backend | HTTP POST | `X-GRPMS-WP-Secret` |
| Salvar pedido | WordPress (PHP, servidor) | Backend | HTTP POST | `X-GRPMS-WP-Secret` |
| Sincronização de comissões (NAO VERIFICADO: `/api/checkout/save-commissions` está na whitelist, grpms-backend-client.php:35, mas sem rota no backend e sem chamador; a gravação de comissões ocorre dentro de `save-order`, backend/fastapi_app/services/checkout/order_service.py:156) | WordPress (PHP, servidor) | Backend | HTTP POST | `X-GRPMS-WP-Secret` |
| Painéis (dados) | Navegador/painéis (VERIFICADO: seller/affiliate chamam o backend direto, frontend/includes/seller/grpms-seller-dashboard.php:171; dashboard principal e galeria passam por proxy PHP, grpms-dashboard.php:152, grpms-gallery.php:177) | Backend | HTTP GET | JWT Bearer (token emitido por `/api/token/from-wp`) |
| Rastreio/histórico de envio EasyPost | WordPress | Backend | HTTP | Identidade de vendedor assinada por HMAC (ver 6.1) |
| Webhook de pedido WooCommerce | WooCommerce | Backend | HTTP POST | Assinatura `x-wc-webhook-signature` (HMAC-SHA256), padrão desligado (ver 6.2) |

**Cabeçalho `X-GRPMS-WP-Secret`:** enviado por `grpms_backend_http()` somente para as seis rotas permitidas abaixo (`includes/api/grpms-backend-client.php:33-35`; header em :46-48). Não é enviado "em todas as requisições".

**Rotas permitidas para WordPress:** `grpms-backend-client.php:33-35`
- `/api/token/from-wp`
- `/api/easypost/calculate-shipping`
- `/api/easypost/validate-selection`
- `/api/checkout/validate-affiliate`
- `/api/checkout/save-order`
- `/api/checkout/save-commissions`

### 6.1. Identidade de vendedor assinada (HMAC) — segundo limite de autenticação

`fastapi_app/auth/wordpress_bridge.py:18-35` (`require_vendor_identity`; VERIFICADO) exige os cabeçalhos `X-GRPMS-Vendor-Id`, `X-GRPMS-Ts` e `X-GRPMS-Sig`. A assinatura é HMAC-SHA256, com a chave `GRPMS_WP_IDENTITY_KEY` (mínimo 32 caracteres, senão 503), sobre `MÉTODO|caminho|query-ordenada|vendor|ts`. O timestamp tolera ±60 s. É usado por `routers/checkout/easypost_router.py` (importação em :3; dependência em rotas como track, tracker status e shipment history, :89, :105, :127). O lado WordPress que gera essa assinatura: NAO VERIFICADO (não localizado nesta revisão).

### 6.2. Webhook assinado de pedidos WooCommerce (padrão desligado)

Existe `POST /webhooks/woocommerce/order` (`fastapi_app/routers/webhooks/woocommerce_order.py`, docstring "Default-off signed WooCommerce webhook endpoint"; registrado em `main.py:24`; linha corrigida). Comportamento no código:
- Só responde se `GRPMS_AUTO_COMPLETE` for verdadeiro (`services/checkout/wc_webhook.py:21-22`); caso contrário retorna 404.
- Verifica assinatura HMAC-SHA256 base64 do corpo com `GRPMS_WC_WEBHOOK_SECRET` (`wc_webhook.py:55-70`); só processa o tópico `order.updated` (:78); limite de corpo de 1 MiB.
- Serviços relacionados: `wc_outbox_worker.py` (um lote de outbox, "No scheduler is installed"; envio exige `GRPMS_WC_SEND_ENABLED`, `GRPMS_AUTO_COMPLETE` e `GRPMS_ENV=lab`, :128-130) e `auto_complete.py` ("Unwired, default-off completion of a confirmed payment").
- Estado em produção/homologação, registro do webhook no WooCommerce e valores das flags: NAO VERIFICADO.
- O comportamento "503/desativado" da documentação de implantação refere-se a **webhooks EasyPost** e compra de etiquetas (`easypost_router.py:85`, :190-192), não a este endpoint.

### 6.3. Endpoints de token aposentados

(VERIFICADO token_router.py:13-16, :18-42; "Clientes móveis dependentes do login antigo são afetados" é da documentação, backend docs/CLOUD-DEPLOYMENT.md:10: NAO VERIFICADO no código.)

`POST /api/token` e `POST /api/token/login` retornam **410** (`token_router.py:13-16`). Apenas `/api/token/from-wp` (:18-42, protegido por `require_wordpress`) emite token. Clientes móveis dependentes do login antigo são afetados (backend docs/CLOUD-DEPLOYMENT.md:10).

---

## 7. Dependências Externas

### 7.1. Gateways de Pagamento
- **Stripe:** provedor de pagamento da plataforma, usado via plugin do WooCommerce (informação do PO). Plugin, versão e configuração: NAO VERIFICADO (fora destes repositórios; WooPayments é só hipótese de documentação). Nenhuma chamada Stripe nos repositórios; o backend lê metadados `_wcpay_*` (financial_fee_service.py:8-20) e o `transaction_id` do webhook de pedido do WooCommerce
- **PayPal:** Integrado via WooCommerce PayPal Payments: NAO VERIFICADO (plugin externo; nenhuma chamada à API PayPal; o frontend guarda o e-mail PayPal para repasses, grpms-paypalset.php:20, e filtra moedas por id de gateway, grpms-checkout.php:1097-1125)

### 7.2. Transportadoras
- **EasyPost:** API para cotação de frete (VERIFICADO easypost_router.py:141, easypost_service.py:10)
  - Modo teste: `EASYPOST_TEST_KEY`
  - Modo produção: `EASYPOST_LIVE_KEY`
  - Compra de etiquetas: `EASYPOST_BUY_LIVE=0` (VERIFICADO no exemplo .env.example:13 e no código easypost_service.py:43; desativado, easypost_router.py:83-85)

### 7.3. Serviços de Taxas de Câmbio
- **frankfurter.dev:** API pública para conversão de moedas (VERIFICADO frontend/grpms-main.php:1145-1176; cache em :1221)
  - Endpoints: `/v2/rate/` e `/v1/latest`
  - Fallback: floatrates.com
  - Cache: 24 horas (DAY_IN_SECONDS)

---

## 8. Maturidade e Status

| Componente | Status | Observações |
|------------|--------|-------------|
| Plugin GRP-MultiSeller | Versão 1.1.0 (VERIFICADO frontend/grpms-main.php:5); a branch é de **homologação** | Não instalar na VM antiga; "não aprovar produção ainda" (frontend docs/CLOUD-DEPLOYMENT.md:3-4, :20). Status de produção NAO VERIFICADO |
| Backend FastAPI | Homologação | Branch cloud/secure-integration |
| Integração WordPress ↔ Backend | Implementada | Via segredo compartilhado |
| Frete (EasyPost) | Homologação | Modo teste |
| Comissões em cascata | Desativado | Aguardando autorização |
| Webhooks | EasyPost: desativado (503). WooCommerce order: implementado, padrão desligado (`GRPMS_AUTO_COMPLETE`) | Ver 6.2 |
| Afiliados | Validação de afiliado implementada no backend (checkout_router.py); YITH Affiliates: plugin externo, NAO VERIFICADO | - |
| Multi-moeda | Parcial | Conversão via API externa |

**UNVERIFIED:** Status exato de produção do backend (necessita confirmação do time de infraestrutura)

---

## 9. Padrões e Convenções

### 9.1. Nomenclatura
- **Prefixo de funções:** `grpms_*` (frontend/grpms-main.php, grpms-backend-client.php)
- **Prefixo de classes:** `GRPMS_` (frontend/grpms-main.php:~1580-1590, instanciações de classes)
- **Prefixo de opções:** `grpms_` (VERIFICADO: opções `grpms_platform_commission`, frontend/includes/admin/platform/grpms-platform.php:42; `grpms_fx_last_valid_*`, frontend/grpms-main.php:1142)
- **Namespace REST:** `grpms/v1` (VERIFICADO grpms-rest-api.php:6-8)

### 9.2. Arquitetura de Arquivos
- **Frontend:** Organizado por funcionalidade em `includes/{admin,affiliation,api,checkout,dash,dashboard,financial,gallery,general,pwa,profile,products,registry,seller,training}/`
- **Backend:** Organizado por domínio em `fastapi_app/{auth,routers,services,utils}/`

### 9.3. Estilo de Código
- **Frontend:** Padrões WordPress (GRPMS_ prefix, hooks, filters)
- **Backend:** PEP 8, type hints, Pydantic models (Pydantic VERIFICADO; PEP 8 e type hints parciais, ex.: `str | None` em wordpress_bridge.py:10; conformidade geral NAO VERIFICADA)
- **CSS:** Arquivos externos apenas, sem `<style>` inline ou atributos `style` (README-FE.md:254-256)
- **JS:** Controla estados visuais por classes, regras no CSS (README-FE.md:257)

---

## 10. Ambientes

| Ambiente | URL | Branch | Status |
|-----------|-----|--------|--------|
| Produção (antigo) | tuzzello.com | UNVERIFIED | - |
| Novo (homologação GCP) | novo.tuzzello.com | cloud/secure-integration | Homologação; "não aprovar produção ainda" (frontend docs/CLOUD-DEPLOYMENT.md:3-4, :20). "Ativo" NAO VERIFICADO |
| Backend (novo) | nf-novo.tuzzello.com | cloud/secure-integration | Homologação (backend docs/CLOUD-DEPLOYMENT.md:3, VM lptaf01pyt01) |
| Homologação | UNVERIFIED | UNVERIFIED | UNVERIFIED |
| Desenvolvimento | Local | main | Ativo |

---

## 11. Referências

- **Frontend:** /home/gilberto/Projects/Tuzzello-AI-Lab/frontend
- **Backend:** /home/gilberto/Projects/Tuzzello-AI-Lab/backend
- **Documentação de implantação:** frontend/docs/CLOUD-DEPLOYMENT.md (VM web) e backend/docs/CLOUD-DEPLOYMENT.md (VM backend, outro arquivo)
- **Processo de conclusão automática:** backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md
