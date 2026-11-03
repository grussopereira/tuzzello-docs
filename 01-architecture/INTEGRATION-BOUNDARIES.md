> **STATUS DO DOCUMENTO: PARCIAL** (revisado em 2026-11-03 após validação ARCH-VALIDATE-01 / ARCH-FIX-01). Citações de linha corrigidas contra o HEAD de frontend e backend; o modelo de implantação descrito é o evidenciado pelas docs de implantação (VMs, não contêineres). Itens não confirmáveis no código estão marcados **NAO VERIFICADO**: Nginx, bloqueio público de rotas, status de produção de qualquer componente, Stripe/WooPayments, PayPal, Podman, YITH Affiliates, topologia/IPs, cron (apenas em README-FE.md), instalação do MU-Plugin de mídia. Flags com padrão desligado (`GRPMS_COMMISSION_WATERFALL`, `EASYPOST_BUY_LIVE`, `GRPMS_AUTO_COMPLETE`, `GRPMS_WC_SEND_ENABLED`) estão descritas como tal.
> **ARCH-FIX-04 (2026-11-03) — Stripe e systemd:** (a) Stripe (ARCH-FIX-05): segundo o Product Owner (informação do proprietário, aceita como declarada), o Stripe é o provedor de pagamento da plataforma, usado por meio de um plugin do WooCommerce. O mecanismo dentro do WordPress (qual plugin, versão e configuração) é **NAO VERIFICADO**: não está nestes repositórios. Nenhuma chamada à API Stripe, PaymentIntent, Checkout Session, webhook Stripe ou verificação de assinatura existe em `frontend/` e `backend/fastapi_app` neste HEAD (apenas fixtures em frontend/tests/payout-approval-guard.php:367,395); o checkout guarda o identificador genérico de método de pagamento do WooCommerce e o backend guarda o `transaction_id` recebido do webhook de pedido do WooCommerce. O ponto de integração é o WooCommerce, não o backend. Referências de configuração fora do código (somente nomes de arquivo e linhas; nenhum conteúdo aberto ou registrado): backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (drop-in `stripe.conf`) e :24 (arquivo `backend/infrastructure/secrets/stripe.env`, referência de arquivo apenas, ausente neste workspace); frontend/docs/REGISTRO-ERROS-CORRECOES.md:579. (b) systemd: não consta em CLOUD-DEPLOYMENT.md (frontend e backend); consta em backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (alegação do repositório lab, conflitante com a decisão do PO `grpms-backend`: serviço de usuário `tuzzello-backend`, **NAO VERIFICADO**) e frontend/docs/REGISTRO-ERROS-CORRECOES.md:579; nome de serviço: decisão do PO = `grpms-backend`; a citação `tuzzello-backend` (PROCESSO-AUTOMATICO-CONCLUSAO.md:22) é alegação do repositório conflitante, NAO VERIFICADO; código/implantação: **NAO VERIFICADO**. (c) Citações de arquivo usam caminho completo a partir da raiz do repositório (`frontend/…`, `backend/…`).
> **ARCH-FIX-06 (2026-11-03) — Nome do serviço:** o nome do serviço de backend foi decidido pelo Product Owner como `grpms-backend` (decisão vinculante). O estado da unidade systemd local é **NAO VERIFICADO** (o barramento de usuário estava inacessível). A alegação do repositório lab `backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22` (serviço de usuário `tuzzello-backend`) conflita com a decisão do PO e permanece citada apenas como alegação: **NAO VERIFICADO**.

> **ARCH-FIX-02 (2026-11-03):** releitura contra o código de frontend e backend concluída; linhas corrigidas onde divergiam e toda afirmação não confirmável marcada **NAO VERIFICADO** no próprio texto. O documento permanece PARCIAL: nenhuma afirmação sobre plugins externos, produção ou topologia foi confirmada.

> **ARCH-FIX-03 (2026-11-03):** as notas de seção das matrizes 1.1–1.3, 2 e 3.x foram substituídas por marcadores por afirmação. Convenção: **VERIFICADO (arquivo:linha)** = confirmado no código de `frontend/includes/*`, `frontend/grpms-main.php` ou `backend/fastapi_app` neste HEAD; **NAO VERIFICADO** = não confirmado por código. Afirmações sobre WordPress core, WooCommerce, WCFM, YITH Affiliates, Stripe/WooPayments e PayPal descrevem plugins externos e são NAO VERIFICADAS, salvo quando o código destes repositórios os referencia (a referência é citada). Status: **PARCIAL**.

# Limites de Integração entre WordPress, WooCommerce, WCFM e Backend

> **Propósito:** Documentar as responsabilidades e limites entre WordPress, WooCommerce, WCFM e o backend FastAPI. **Data:** 2026-11-03. Descreve o código no HEAD dos repositórios.

---

## 1. Visão Geral dos Componentes e Suas Responsabilidades

### 1.1. WordPress Core

| Componente | Responsabilidade | Limites | Interação com Tuzzello |
|------------|------------------|---------|------------------------|
| **Núcleo** | Gerenciamento de usuários, posts, pages, taxonomias | - | Estendido via hooks e plugins (comportamento padrão do WordPress, sem código local; NAO VERIFICADO) |
| **Roles & Capabilities** | Controle de acesso base | Definição de roles | Tuzzello adiciona roles customizadas (tuzzello_student) |
| **Sessão** | Autenticação de usuários | Cookies, nonce | Tuzzello usa sessão para gerar tokens JWT |
| ** REST API** | API REST nativa | Endpoints padrão | Tuzzello registra endpoints customizados |

> **Marcadores por afirmação (1.1):**
> - Núcleo, Sessão (cookies/nonce), REST nativa e "Hooks e filters": NAO VERIFICADO (comportamento do WordPress, sem código local). 
> - "Tuzzello adiciona roles customizadas (tuzzello_student)": VERIFICADO frontend/grpms-main.php:13-21.
> - "Tuzzello usa sessão para gerar tokens JWT": VERIFICADO (`is_user_logged_in()` e `get_current_user_id()` em grpms-backend-client.php:74-80; nonce `grpms_dashboard_token` em :93).
> - "Tuzzello registra endpoints customizados": VERIFICADO grpms-rest-api.php:908- (namespace `grpms/v1`, :6-8).
> - "Gerenciamento de usuários [só WordPress]": **parcialmente contradito**: o plugin cria usuários com `wp_insert_user` (grpms-registry.php:212; class-grpms-training-enrollment.php:126) e ativa/desativa contas (grpms-registry.php:487, :967).

**Responsabilidades do WordPress:**
- Gerenciamento de usuários e autenticação
- Gerenciamento de conteúdo (posts, pages)
- Sistema de templates
- Hooks e filters para extensão

**Limites:**
- Não gerencia lógicas de negócios específicas do marketplace (NAO VERIFICADO, e contradito em parte: a lógica de marketplace está no plugin GRP-MultiSeller dentro do WordPress, ver 1.4)
- Não acessa backend FastAPI diretamente (NAO VERIFICADO: o cliente `grpms_backend_http` roda dentro do WordPress/plugin, grpms-backend-client.php:27; "diretamente" refere-se apenas ao navegador)
- Não lida com cálculo de frete ou comissões (NAO VERIFICADO para o WordPress core; no plugin, o frete é obtido do backend, VERIFICADO grpms-guest-shipping.php:107, e a percentagem de comissão da plataforma é gravada no WordPress, VERIFICADO frontend/includes/admin/platform/grpms-platform.php:42)

**Fontes:**
- VERIFICADO frontend/grpms-main.php:13-21 (registro de role tuzzello_student; `Version: 1.1.0` em :5)
- README-FE.md:13 (WordPress 6.0+ requerido; verificado na documentação; versão em runtime NAO VERIFICADA)

---

### 1.2. WooCommerce

| Funcionalidade | Responsabilidade | Limites | Integração Tuzzello |
|----------------|------------------|---------|---------------------|
| **Catálogo** | Gerenciamento de produtos, categorias, atributos | Produtos físicos/digitais | Tuzzello estende com multi-moeda |
| **Carrinho** | Gerenciamento de carrinho de compras | Sessão do usuário | Tuzzello valida carrinho antes de checkout |
| **Checkout** | Processo de checkout | Gateways de pagamento | Tuzzello customiza fluxo de checkout |
| **Pedidos** | Gerenciamento de pedidos | Status, itens, valores | Tuzzello sincroniza com backend |
| **Clientes** | Gerenciamento de clientes | Dados de cliente | Tuzzello adiciona campos customizados |
| **Relatórios** | Relatórios de vendas | Dados do WooCommerce | Tuzzello adiciona relatórios customizados |
| **Configurações** | Configurações gerais | Opções do plugin | Tuzzello adiciona configurações |

**Marcadores por afirmação (1.2):** "Tuzzello estende com multi-moeda" VERIFICADO (frontend/grpms-main.php:807-1123); "valida carrinho antes de checkout" VERIFICADO em parte (revalidação do carrinho contra a oferta de afiliado, grpms-guest-shipping.php:208-215); "customiza fluxo de checkout" VERIFICADO (grpms-checkout.php, 51 hooks); "sincroniza com backend" VERIFICADO (save-order, grpms-checkout.php:2029); "relatórios customizados" VERIFICADO em parte (grpms-orders-history.php:603); "adiciona campos customizados a clientes" e "adiciona configurações": NAO VERIFICADO. A coluna "Responsabilidade/Limites" descreve o WooCommerce externo: NAO VERIFICADO.

**Responsabilidades do WooCommerce (todas NAO VERIFICADAS: plugin externo):**
- Gerenciamento completo de produtos
- Processo de compra (carrinho → checkout → pedido)
- Gateways de pagamento: Stripe é o provedor segundo o PO, via plugin do WooCommerce (plugin, versão e configuração NAO VERIFICADOS); PayPal: NAO VERIFICADO. O ponto de integração é o WooCommerce, não o backend. Nos repositórios: nenhuma chamada à API Stripe ou PayPal; o frontend detecta ids de gateway `paypal`/`ppcp` (grpms-checkout.php:1105-1112) e o backend lê metadados `_wcpay_*` do WooPayments (financial_fee_service.py:8-20)
- Cálculo de impostos
- Gerenciamento de estoque

**Limites (NAO VERIFICADOS: plugin externo):**
- Não gerencia multi-vendedores (requer WCFM)
- Não gerencia comissões entre partes
- Não gerencia afiliação (requer YITH Affiliates)
- Frete limitado sem plugins adicionais

**Fontes:**
- README-FE.md:14 (WooCommerce 7.0+ requerido; verificado)
- README-FE.md:15-16 (WooPayments 9.0+ e WooCommerce PayPal Payments 3.1.0+ requeridos; verificado; funcionamento real NAO VERIFICADO)

---

### 1.3. WCFM (WC Frontend Manager)

| Funcionalidade | Responsabilidade | Limites | Integração Tuzzello |
|----------------|------------------|---------|---------------------|
| **Multi-vendedor** | Permite múltiplos vendedores no marketplace | Cada vendedor gerencia seus produtos | Tuzzello estende com validações |
| **Store Manager** | Dashboard para vendedores | Visão limitada ao vendedor | Tuzzello adiciona funcionalidades |
| **Comissões** | Cálculo de comissões | Comissões básicas | Tuzzello substitui por lógica custom |
| **Withdrawals** | Saques para vendedores | Processo de saque | Tuzzello customiza processo |
| **Membership** | Planos de assinatura | - | NAO VERIFICADO |
| **Product Manager** | Gerenciamento de produtos | Interface para vendedores | Tuzzello força produtos virtuais |

**Marcadores por afirmação (1.3):** "Tuzzello substitui [comissões] por lógica custom": VERIFICADO em parte (percentagem da plataforma em `grpms_platform_commission`, frontend/includes/admin/platform/grpms-platform.php:42; comissão calculada no backend, backend/fastapi_app/services/checkout/order_service.py:156); "Tuzzello força produtos virtuais": VERIFICADO (frontend/grpms-main.php:1264-1321); "Tuzzello estende com validações / adiciona funcionalidades / customiza processo de saque": VERIFICADO em parte (saques tratados em grpms-payment.php:724, 773); "Membership", comissões básicas e demais responsabilidades do WCFM: NAO VERIFICADO (plugin externo).

**Responsabilidades do WCFM (NAO VERIFICADAS: plugin externo):**
- Gerenciamento de vendedores (wcfm_vendor role)
- Dashboard do vendedor
- Cálculo básico de comissões
- Processo de saque
- Gerenciamento de produtos por vendedor

**Limites:**
- Não gerencia afiliação
- Comissões limitadas
- Não gerencia multi-moeda
- Não integra com backend externo

**Fontes:**
- WCFM não consta da lista de requisitos de README-FE.md:13-26 (NAO VERIFICADO como requisito declarado); o código usa hooks e role do WCFM (VERIFICADO frontend/grpms-main.php:1310, :1320; role `wcfm_vendor` grpms-backend-client.php:95)
- VERIFICADO frontend/grpms-main.php:1264-1321 (forçar produtos virtuais para vendedores; função em :1264)

**Roles do WCFM:**
- `wcfm_vendor`: Role principal do vendedor (VERIFICADO uso em frontend/grpms-main.php:907 `in_array('wcfm_vendor', ...)` e :1266; a definição da role pertence ao plugin WCFM: NAO VERIFICADO)

---

### 1.4. Plugin GRP-MultiSeller

**Responsabilidades:**
- **Controle de Acesso:** Sistema de capabilities customizadas (VERIFICADO includes/admin/access/, class-grpms-access-capability.php:6)
- **Integração Backend:** Cliente HTTP para FastAPI (VERIFICADO includes/api/grpms-backend-client.php:27)
- **API REST:** Endpoints customizados (VERIFICADO includes/api/grpms-rest-api.php:908-)
- **Afiliação:** Gerenciamento de afiliados (VERIFICADO includes/affiliation/grpms-affiliation.php:75, :89)
- **Checkout:** Lógica customizada de checkout (VERIFICADO includes/checkout/grpms-checkout.php:848-984, :2029)
- **Comissões:** **INCORRETO** como escrito: `includes/grpms-commissions.php` tem 11 linhas e só imprime `the_content()` (VERIFICADO). No plugin, VERIFICADO apenas a configuração da percentagem da plataforma (frontend/includes/admin/platform/grpms-platform.php:42) e a comissão por produto do afiliado (grpms-affiliation.php:197); o cálculo está no backend (backend/fastapi_app/services/checkout/order_service.py:156)
- **Dashboard:** Painéis customizados (VERIFICADO includes/dashboard/grpms-dashboard.php:13-76)
- **Multi-moeda:** Suporte a múltiplas moedas (VERIFICADO frontend/grpms-main.php:807-1123; funções de moeda, câmbio e atribuição de moeda)
- **Frete:** Integração com EasyPost via backend (VERIFICADO grpms-guest-shipping.php:107, :141; easypost_service.py:353)
- **Produtos:** Validações e extensões (VERIFICADO includes/products/grpms-products.php:2184, product-text-limits.php:5)
- **Vendedores:** Funções utilitárias para vendedores (VERIFICADO includes/seller/grpms-seller.php, grpms-shipping-config.php, grpms-seller-store-config.php)
- **Treinamento:** Área de treinamento (VERIFICADO includes/training/grpms-training.php:314-315)
- **PWA:** Suporte a Progressive Web App (VERIFICADO includes/pwa/grpms-pwa.php:315 manifest, :358 service worker)
- **Não listados antes e VERIFICADOS:** Registro/ativação de usuários (grpms-registry.php:140, :487), KYC (grpms-kyc.php:285, profile-kyc-files.php), TOTP (class-grpms-totp-login-gate.php:6), saques/repasses (grpms-payment.php:724), galeria (grpms-gallery.php:45), aprovação de produtos e pedidos (grpms-products-approval.php:577, grpms-approve_orders.php:9)

**Limites:**
- Acessa o BD do WordPress via `$wpdb` (ex.: includes/api/grpms-rest-api.php, includes/general/grpms-general.php); a frase anterior "não acessa BD diretamente" era imprecisa; a tabela 3.10 mostra consultas diretas
- Não executa lógica de negócios complexa (delegado ao backend): **contradito pelo código**; há lógica de moeda, câmbio (frontend/grpms-main.php:1126), checkout (grpms-checkout.php), repasses (grpms-payment.php) e aprovação de pedidos (grpms-approve_orders.php:171) no plugin
- Não gerencia usuários (delegado ao WordPress): **contradito pelo código**; o plugin cria usuários (`wp_insert_user`, grpms-registry.php:212) e altera status (grpms-registry.php:487, :967); a autenticação em si permanece no WordPress (NAO VERIFICADO)
- Não processa pagamentos de compradores (delegado ao WooCommerce): NAO VERIFICADO (plugin externo), mas consistente com o código: `includes/financial/grpms-payment.php` (2869 linhas, lido) trata solicitações de repasse/saque de vendedores e afiliados (VERIFICADO grpms-payment.php:724, :773), não cobrança de comprador, e não chama API de pagamento

**Fontes:**
- VERIFICADO frontend/grpms-main.php (plugin principal; ao menos até a linha 1695)
- VERIFICADO frontend/grpms-main.php:1480-1508 (carregamento de módulos)

---

### 1.5. Backend FastAPI

**Responsabilidades:**
- **Autenticação:** Geração de tokens JWT (VERIFICADO routers/token_router.py:18-42)
- **Validação:** Validação de links de afiliados (VERIFICADO routers/checkout/checkout_router.py:27-43; checkout_service.py:41)
- **Frete:** Cálculo de frete via EasyPost (VERIFICADO routers/checkout/easypost_router.py:141; easypost_service.py:353). Também VERIFICADO: frete por zona do vendedor sem EasyPost (shipping_router.py:15, shipping_service.py:5)
- **Pedidos:** Processamento de pedidos (VERIFICADO backend/fastapi_app/services/checkout/order_service.py:504)
- **Comissões:** Cálculo de comissões (VERIFICADO backend/fastapi_app/services/checkout/commission_math.py: `compute_commissions`, :4; `save_commissions` em backend/fastapi_app/services/checkout/order_service.py:156; o waterfall só é usado com `GRPMS_COMMISSION_WATERFALL`, :133-140, :199)
- **Dashboard:** Dados para dashboards (VERIFICADO routers/dashboard_main.py:25-33, seller_dashboard_router.py:11, affiliate_dashboard_router.py:8)
- **Galeria:** Gerenciamento de galeria de produtos (VERIFICADO apenas leitura/busca: product_gallery.py:18, product_gallery_search.py:16; "gerenciamento" NAO VERIFICADO)
- **Webhooks:** Processamento do webhook assinado de pedidos WooCommerce (VERIFICADO wc_webhook.py:21, :52; padrão desligado)

**Limites:**
- Não acessa WordPress admin ou interfaces — NAO VERIFICADO
- Não lida com sessão do WordPress diretamente — NAO VERIFICADO (recebe `user_id` do WordPress, token_router.py:10-11)
- Não processa requests do navegador diretamente (apenas via WordPress) — NAO VERIFICADO: CORS permite origem de navegador (main.py:26-32) e rotas de dashboard usam JWT, então o navegador pode chamar o backend diretamente
- Acessa BD WordPress diretamente (tabelas wp_*) (VERIFICADO token_router.py:24, checkout_service.py:9, backend/fastapi_app/services/checkout/order_service.py:50)

**Fontes:**
- fastapi_app/main.py:1-48 (aplicação principal)
- fastapi_app/utils/database.py:8-30 (conexão com BD)

---

## 2. Diagramas de Responsabilidades

> **Marcadores por elemento (diagramas 2.1 e 2.2, ARCH-FIX-03):** conceituais. Camadas Navegador → WordPress → Backend: VERIFICADO em parte (fluxo por `grpms_backend_http`, grpms-backend-client.php:27; JWT para dashboards, :74). "MySQL": driver `pymysql` VERIFICADO (database.py:20), MySQL em produção NAO VERIFICADO. Stripe e PayPal: NAO VERIFICADO (nenhuma chamada nos repositórios). EasyPost ← Backend: VERIFICADO (easypost_service.py:10). **Frankfurter ← Backend: INCORRETO**: a consulta de câmbio é feita pelo WordPress (frontend/grpms-main.php:1145-1176); `backend/fastapi_app` não referencia Frankfurter nem floatrates (busca negativa). A seta "WordPress → Backend → MySQL" do diagrama 2.2 é ilustrativa: WordPress usa `$wpdb` diretamente (VERIFICADO grpms-rest-api.php:92) e o backend acessa o mesmo banco por conta própria (VERIFICADO database.py:8-30).

### 2.1. Diagrama de Camadas

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        NAVEGADOR (Cliente)                                │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐            │
│  │   HTML/CSS/JS   │  │   Cookies        │  │   JWT Token     │            │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘            │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      WORDPRESS (Parcialmente Confiável)                  │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    WORDPRESS CORE                                │   │
│  │  Usuários │ Posts │ Pages │ Taxonomias │ Sessão │ REST API    │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    WOOCOMMERCE                                  │   │
│  │  Catálogo │ Carrinho │ Checkout │ Pedidos │ Clientes        │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    WCFM                                         │   │
│  │  Multi-vendedor │ Store Manager │ Comissões │ Withdrawals    │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    GRP-MULTISELLER PLUGIN                        │   │
│  │  Acesso │ API │ Afiliação │ Checkout │ Comissões │ Dashboard   │   │
│  │  Multi-Moeda │ Frete │ Produtos │ Vendedores │ Treinamento    │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
            ┌───────────┐   ┌───────────┐   ┌───────────┐
            │ MySQL     │   │ Stripe via│   │ PayPal    │
            │ (wp_*)    │   │ WooComm.NV│   │           │
            └───────────┘   └───────────┘   └───────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      BACKEND FASTAPI (Confiável)                        │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Autenticação │ Validação │ Frete │ Pedidos │ Comissões       │   │
│  │  Dashboard │ Galeria │ Webhooks │ EasyPost Integration       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
            ┌───────────┐   ┌───────────┐   ┌───────────┐
            │ MySQL     │   │ EasyPost  │   │ Frankfurter│
            │ (wp_*)    │   │           │   │           │
            └───────────┘   └───────────┘   └───────────┘
```

---

### 2.2. Diagrama de Fluxo de Dados

```
┌─────────────┐    ┌─────────────┐    ┌─────────────────┐    ┌─────────────┐
│   Navegador  │───▶│  WordPress   │───▶│  FastAPI Backend  │───▶│   MySQL     │
│   (Cliente)  │    │   (PHP)      │    │   (Python)        │    │   (wp_*)    │
└─────────────┘    └─────────────┘    └─────────────────┘    └─────────────┘
       │               │                     │                      │
       │  HTML/JS      │  Session            │  X-GRPMS-WP-Secret   │
       │  (Cookies)    │  (WordPress)        │  (Autenticação)     │
       │               │                     │                      │
       │               │  Permissões         │                      │
       │               │  (Capabilities)     │                      │
       │               │                     │                      │
       └─────────────────────────────────────────────────────────────────┘
```

---

## 3. Limites de Responsabilidade

> **Marcadores por célula (matrizes 3.1–3.10, ARCH-FIX-03):** as matrizes (✅/❌) são síntese do arquiteto. Regra aplicada: células das colunas **WooCommerce, WCFM e YITH Affiliates** = NAO VERIFICADAS (plugins externos). Células das colunas **GRP-MultiSeller** e **Backend** = VERIFICADAS somente quando há linha citada em "Fontes" ou nas correções abaixo; as demais, NAO VERIFICADAS. Células de **WordPress** (core) = NAO VERIFICADAS. Correções feitas na leitura: 3.1 "Criação de usuários [GRP-MultiSeller ❌]" é **contradita** (grpms-registry.php:212 usa `wp_insert_user`); 3.1 "Validação de tokens JWT [Backend ✅]" VERIFICADA (jwt_rs256.py:25, dependencies.py:9-17, usado nos routers de dashboard/galeria); 3.5 "Cálculo de comissões [GRP-MultiSeller ✅]" **NAO VERIFICADO** (ver abaixo); 3.7 "Dashboard [Backend ✅]" VERIFICADO (rotas JWT).

### 3.1. Gerenciamento de Usuários

| Responsabilidade | WordPress | WooCommerce | WCFM | GRP-MultiSeller | Backend |
|------------------|-----------|-------------|------|-----------------|---------|
| Criação de usuários | ✅ | ❌ | ❌ | ❌ | ❌ |
| Autenticação | ✅ | ❌ | ❌ | ❌ | ❌ |
| Sessão | ✅ | ❌ | ❌ | ❌ | ❌ |
| Roles (administrator, editor, etc.) | ✅ | ❌ | ❌ | ❌ | ❌ |
| Role wcfm_vendor | ❌ | ❌ | ✅ | ❌ | ❌ |
| Role tuzzello_student | ❌ | ❌ | ❌ | ✅ | ❌ |
| Capabilities customizadas | ❌ | ❌ | ❌ | ✅ | ❌ |
| Geração de tokens JWT | ❌ | ❌ | ❌ | ❌ | ✅ |
| Validação de tokens JWT | ❌ | ❌ | ❌ | ❌ | ✅ |

**Fontes:**
- VERIFICADO frontend/grpms-main.php:13-21 (registro de role tuzzello_student)
- VERIFICADO includes/admin/access/ (sistema de capabilities)
- VERIFICADO fastapi_app/routers/token_router.py:18-42 (geração de tokens)
- VERIFICADO "Validação de tokens JWT" no backend: `verify_token` jwt_rs256.py:25, usado por `get_current_user` (dependencies.py:9-17) nas rotas de dashboard, galeria e mobile

---

### 3.2. Gerenciamento de Produtos

| Responsabilidade | WordPress | WooCommerce | WCFM | GRP-MultiSeller | Backend |
|------------------|-----------|-------------|------|-----------------|---------|
| Criação de produtos | ❌ | ✅ | ✅ (frontend) | ❌ | ❌ |
| Edição de produtos | ❌ | ✅ | ✅ (frontend) | ❌ | ❌ |
| Exclusão de produtos | ❌ | ✅ | ✅ (frontend) | ✅ (admin) | ❌ |
| Categorias e tags | ❌ | ✅ | ✅ | ❌ | ❌ |
| Atributos | ❌ | ✅ | ✅ | ❌ | ❌ |
| Estoque | ❌ | ✅ | ✅ | ❌ | ❌ |
| Preços | ❌ | ✅ | ✅ | ✅ (multi-moeda) | ❌ |
| Moeda do produto | ❌ | ❌ | ❌ | ✅ | ❌ |
| Forçar produtos virtuais | ❌ | ❌ | ❌ | ✅ | ❌ |
| Aprovação de produtos | ❌ | ❌ | ❌ | ✅ (admin) | ❌ |
|Galeria de produtos | ❌ | ❌ | ❌ | ✅ | ✅ (backend) |

**Fontes:**
- VERIFICADO frontend/grpms-main.php:1089-1123 (atribuição de moeda ao produto)
- VERIFICADO frontend/grpms-main.php:1264-1321 (forçar produtos virtuais)
- VERIFICADO includes/admin/products_approval/grpms-products-approval.php:577-619
- VERIFICADO includes/admin/delete_products/grpms-product-delete.php:11, :73
- Criação/edição de produtos [GRP-MultiSeller ❌]: **contradito em parte**: o plugin salva e edita produtos (grpms-products.php:2184, `save_product_changes`)
- Galeria [Backend ✅]: VERIFICADO somente leitura/busca (product_gallery.py:18)

---

### 3.3. Processo de Compra

| Responsabilidade | WordPress | WooCommerce | WCFM | GRP-MultiSeller | Backend |
|------------------|-----------|-------------|------|-----------------|---------|
| Carrinho de compras | ❌ | ✅ | ❌ | ❌ | ❌ |
| Checkout | ❌ | ✅ | ❌ | ✅ (custom) | ❌ |
| Validação de dados | ❌ | ✅ | ❌ | ✅ | ✅ |
| Gateways de pagamento | ❌ | ✅ | ❌ | ❌ | ❌ |
| Processamento de pagamento | ❌ | ✅ | ❌ | ❌ | ❌ |
| Criação de pedido | ❌ | ✅ | ❌ | ❌ | ❌ |
| Sincronização de pedido | ❌ | ❌ | ❌ | ❌ | ✅ |
| Validação de afiliado | ❌ | ❌ | ❌ | ❌ | ✅ |
| Cálculo de frete | ❌ | ❌ | ❌ | ❌ | ✅ |
| Salvar pedido no backend | ❌ | ❌ | ❌ | ✅ | ✅ |
| Salvar comissões | ❌ | ❌ | ❌ | ✅ | ✅ |

**Fontes:**
- NAO VERIFICADO: a citação anterior (frontend/grpms-main.php:871-873) não corresponde a validação de afiliado; ver checkout_router.py
- VERIFICADO includes/api/grpms-backend-client.php:33-35 (rotas permitidas)
- VERIFICADO fastapi_app/routers/checkout/checkout_router.py:27-43 (validação de afiliado); chamadores PHP em grpms-checkout.php:1287 e grpms-guest-shipping.php:218
- "Salvar pedido no backend" VERIFICADO (grpms-checkout.php:2029 → backend/fastapi_app/routers/checkout/order_router.py:11)
- "Salvar comissões": NAO VERIFICADO como rota; `/api/checkout/save-commissions` consta na whitelist (grpms-backend-client.php:35) mas não existe rota nem chamador; a função `save_commissions` roda dentro de `insert_order_into_main_tables` (backend/fastapi_app/services/checkout/order_service.py:156, :504)
- "Cálculo de frete [Backend ✅]": VERIFICADO (easypost_router.py:141, shipping_router.py:15)
- "Sincronização de pedido [Backend ✅]": VERIFICADO como gravação do pedido (backend/fastapi_app/services/checkout/order_service.py:504); sincronização contínua NAO VERIFICADA

---

### 3.4. Afiliação

| Responsabilidade | WordPress | WooCommerce | WCFM | YITH Affiliates | GRP-MultiSeller | Backend |
|------------------|-----------|-------------|------|----------------|-----------------|---------|
| Cadastro de afiliados | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| Gerenciamento de afiliados | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Links de afiliados | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Validação de links | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Comissões de afiliados | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Dashboard do afiliado | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |

**Fontes:**
- VERIFICADO includes/affiliation/ (links de afiliado: grpms-affiliation.php:75, :89; dashboard: grpms-affiliate-dashboard.php:8)
- VERIFICADO fastapi_app/routers/affiliate_dashboard_router.py:8
- VERIFICADO fastapi_app/routers/checkout/checkout_router.py:27-43 (validação)
- Coluna YITH Affiliates ✅ (cadastro, links, comissões, dashboard): NAO VERIFICADO (plugin externo; o código só referencia a role `yith_affiliate`: grpms-affiliation.php:17, checkout_service.py:24)

---

### 3.5. Comissões

| Responsabilidade | WordPress | WooCommerce | WCFM | GRP-MultiSeller | Backend |
|------------------|-----------|-------------|------|-----------------|---------|
| Cálculo de comissões | ❌ | ❌ | ✅ (básico) | ✅ | ✅ |
| Distribuição de comissões | ❌ | ❌ | ✅ (básico) | ✅ | ✅ |
| Comissões em cascata | ❌ | ❌ | ❌ | ❌ | ✅ (desativado) |
| Comissão da plataforma | ❌ | ❌ | ❌ | ✅ | ✅ |
| Comissão do vendedor | ❌ | ❌ | ✅ | ✅ | ✅ |
| Comissão do afiliado | ❌ | ❌ | ❌ | ✅ | ✅ |
| Guarda de reembolso | ❌ | ❌ | ❌ | ❌ | ✅ |

**Fontes:**
- **INCORRETO**: includes/grpms-commissions.php não calcula comissões (11 linhas, imprime `the_content()`, VERIFICADO). Linha "Cálculo de comissões [GRP-MultiSeller ✅]": NAO VERIFICADO; no plugin só há configuração da percentagem da plataforma (frontend/includes/admin/platform/grpms-platform.php:42) e de comissão por produto do afiliado (grpms-affiliation.php:197)
- VERIFICADO backend/fastapi_app/services/checkout/commission_math.py:4 e backend/fastapi_app/services/checkout/order_service.py:156
- "Guarda de reembolso [Backend ✅]": VERIFICADO como código (refund_guard.py:49) porém não ligado a rota (refund_store.py: "Unwired"); efeito em runtime NAO VERIFICADO
- Coluna WCFM ✅ (básico): NAO VERIFICADO (plugin externo)
- backend/.env.example:14-15 (comentário e flag GRPMS_COMMISSION_WATERFALL, padrão desligado; "PO": NAO VERIFICADO, termo não definido)

---

### 3.6. Frete

| Responsabilidade | WordPress | WooCommerce | WCFM | GRP-MultiSeller | Backend |
|------------------|-----------|-------------|------|-----------------|---------|
| Cálculo de frete | ❌ | ✅ (básico) | ❌ | ❌ | ✅ (EasyPost) |
| Configuração de frete | ❌ | ✅ | ❌ | ❌ | ❌ |
| Cotação de frete | ❌ | ❌ | ❌ | ❌ | ✅ |
| Validação de cotação | ❌ | ❌ | ❌ | ❌ | ✅ |
| Compra de etiquetas | ❌ | ❌ | ❌ | ❌ | ✅ (desativado, 503: easypost_router.py:83-85; verificado) |
| Webhooks de frete (EasyPost) | ❌ | ❌ | ❌ | ❌ | ✅ (desativado, 503: easypost_router.py:190-192; verificado) |
| Webhook de pedido WooCommerce | ❌ | ❌ | ❌ | ❌ | ✅ (implementado, padrão desligado: `GRPMS_AUTO_COMPLETE`; HMAC) |

**Fontes:**
- VERIFICADO backend/.env.example:10-13 (configuração EasyPost)
- VERIFICADO fastapi_app/routers/checkout/easypost_router.py (rotas :53-:190)
- frontend docs/CLOUD-DEPLOYMENT.md:22-29 (limitações de frete; documentação, NAO VERIFICADO no código)
- "Configuração de frete [GRP-MultiSeller ❌]": **contradito**: o plugin tem configuração de zonas de frete do vendedor (VERIFICADO grpms-shipping-config.php:130-423), lida pelo backend (shipping_service.py:5-30)
- "Cálculo de frete [WooCommerce ✅ (básico)]": NAO VERIFICADO (plugin externo)

---

### 3.7. Dashboard

| Responsabilidade | WordPress | WooCommerce | WCFM | GRP-MultiSeller | Backend |
|------------------|-----------|-------------|------|-----------------|---------|
| Dashboard principal | ❌ | ❌ | ❌ | ✅ | ✅ |
| Dashboard do vendedor | ❌ | ❌ | ✅ | ✅ | ✅ |
| Dashboard do afiliado | ❌ | ❌ | ❌ | ✅ | ✅ |
| Estatísticas | ❌ | ✅ (básico) | ✅ (básico) | ✅ | ✅ |
| Token de dashboard | ❌ | ❌ | ❌ | ✅ | ❌ |

**Fontes:**
- VERIFICADO includes/dashboard/grpms-dashboard.php:13-76
- VERIFICADO includes/api/grpms-backend-client.php:74-110 (token de dashboard)
- VERIFICADO fastapi_app/routers/dashboard_main.py:25-33
- VERIFICADO fastapi_app/routers/seller_dashboard_router.py:11
- VERIFICADO fastapi_app/routers/affiliate_dashboard_router.py:8
- Colunas WooCommerce/WCFM (estatísticas, dashboard do vendedor): NAO VERIFICADO (plugins externos; o backend lê tabelas, não chama os plugins: dashboard_main_statistics.py:141-143)

---

### 3.8. Multi-Moeda

| Responsabilidade | WordPress | WooCommerce | WCFM | GRP-MultiSeller | Backend |
|------------------|-----------|-------------|------|-----------------|---------|
| Moedas suportadas | ❌ | ❌ | ❌ | ✅ (EUR, USD, BRL, ZAR, XOF) | ❌ |
| Moeda preferida do usuário | ❌ | ❌ | ❌ | ✅ (user_meta) | ❌ |
| Moeda do produto | ❌ | ❌ | ❌ | ✅ | ❌ |
| Símbolo da moeda | ❌ | ❌ | ❌ | ✅ | ❌ |
| Taxa de câmbio | ❌ | ❌ | ❌ | ✅ (APIs externas) | ❌ |
| Conversão de valores | ❌ | ❌ | ❌ | ✅ | ❌ |

**Fontes:**
- VERIFICADO frontend/grpms-main.php:807-817 (moedas suportadas EUR, USD, BRL, ZAR, XOF: :808)
- VERIFICADO frontend/grpms-main.php:1126-1240 (taxa de câmbio)
- VERIFICADO frontend/grpms-main.php:1089-1123 (atribuição de moeda ao produto)
- "Backend ❌" em todas as linhas: VERIFICADO para câmbio (nenhuma referência a Frankfurter no backend); o backend, porém, trata moeda de repasse/comissão em outros pontos: NAO VERIFICADO

---

### 3.9. Acesso e Controle

| Responsabilidade | WordPress | WooCommerce | WCFM | GRP-MultiSeller | Backend |
|------------------|-----------|-------------|------|-----------------|---------|
| Controle de acesso | ✅ (básico) | ❌ | ❌ | ✅ (custom) | ❌ |
| Capabilities customizadas | ❌ | ❌ | ❌ | ✅ | ❌ |
| Validação de acesso | ❌ | ❌ | ❌ | ✅ | ❌ |
| Auditoria de acesso | ❌ | ❌ | ❌ | ✅ | ❌ |
| Autenticação JWT | ❌ | ❌ | ❌ | ❌ | ✅ |
| Validação de JWT | ❌ | ❌ | ❌ | ❌ | ✅ |
| Segredo compartilhado | ❌ | ❌ | ❌ | ✅ (envio) | ✅ (validação) |

**Fontes:**
- VERIFICADO includes/admin/access/ (capability, validator, audit, floor)
- VERIFICADO fastapi_app/auth/ (jwt_rs256.py, wordpress_bridge.py, dependencies.py)
- VERIFICADO includes/api/grpms-backend-client.php:46-48 (envio de segredo)
- VERIFICADO fastapi_app/auth/wordpress_bridge.py:10-15 (validação de segredo)
- "Controle de acesso [WordPress ✅ (básico)]": NAO VERIFICADO (WordPress core)

---

### 3.10. Conexão com Banco de Dados

| Responsabilidade | WordPress | WooCommerce | WCFM | GRP-MultiSeller | Backend |
|------------------|-----------|-------------|------|-----------------|---------|
| Conexão MySQL (NAO VERIFICADO para WooCommerce/WCFM; GRPMS e backend confirmados em código) | ✅ ($wpdb) | ✅ ($wpdb) | ✅ ($wpdb) | ✅ ($wpdb) | ✅ (pymysql, database.py:21) |
| Acesso às tabelas wp_* | ✅ | ✅ | ✅ | ✅ | ✅ |
| Queries diretas | ❌ | ❌ | ❌ | ✅ | ✅ |
| ORM | ❌ | ❌ | ❌ | ❌ | ❌ |

**Fontes:**
- VERIFICADO fastapi_app/utils/database.py:8-30 (conexão backend) e `$wpdb` no plugin (grpms-rest-api.php:92, grpms-general.php)
- Colunas WordPress/WooCommerce/WCFM (`$wpdb`): NAO VERIFICADO (código externo)
- "ORM ❌": VERIFICADO no backend (SQL direto com cursor, ex.: backend/fastapi_app/services/checkout/order_service.py:44-60)

---

## 4. Matriz de Integração

### 4.1. WordPress ↔ WooCommerce

| Tipo | Mechanismo | Direção | Dados | Fontes |
|------|------------|---------|-------|--------|
| Hooks | actions & filters | Bidirecional | Todos | WooCommerce docs |
| REST API | Endpoints | WordPress → WooCommerce | Produtos, pedidos | WooCommerce REST API |
| Functions | WC_* classes | Bidirecional | Todos | WooCommerce codebase |

**Nível de acoplamento:** Alto (WooCommerce é um plugin do WordPress): NAO VERIFICADO. Seções 4.1 e 4.2 (mecanismos de WooCommerce e WCFM) descrevem plataformas externas; cada linha das tabelas é NAO VERIFICADA no código local. Referências no código às APIs do WooCommerce: `WC()`, `wc_get_order` em grpms-checkout.php e grpms-affiliategeneral.php:128 (VERIFICADO)

---

### 4.2. WordPress/WooCommerce ↔ WCFM

| Tipo | Mechanismo | Direção | Dados | Fontes |
|------|------------|---------|-------|--------|
| Hooks | actions & filters | Bidirecional | Vendedores, produtos | WCFM docs |
| Shortcodes | [wcfm_*] | WordPress → WCFM | Interface | WCFM docs |
| Functions | WCFM_* classes | Bidirecional | Vendedores, pedidos | WCFM codebase |

**Nível de acoplamento:** Alto (WCFM estende WooCommerce): NAO VERIFICADO (plugin externo). O código usa os hooks `wcfm_product_manager_data_filter` e `after_wcfm_products_manage_meta_save` (VERIFICADO frontend/grpms-main.php:1310, :1320); o uso de `[wcfm_*]` shortcodes, classes `WCFM_*` e "Bidirecional": NAO VERIFICADO

---

### 4.3. WordPress/WooCommerce/WCFM ↔ GRP-MultiSeller

| Tipo | Mechanismo | Direção | Dados | Fontes |
|------|------------|---------|-------|--------|
| Hooks | actions & filters | Bidirecional | Todos | frontend/grpms-main.php |
| Shortcodes | [grp_*] | WordPress → GRP-MultiSeller | Interface | includes/dashboard/ |
| AJAX | wp_ajax_* | WordPress → GRP-MultiSeller | Operações | includes/admin/access/classes/ |
| REST API | Endpoints | Bidirecional | Dados | includes/api/grpms-rest-api.php |
| HTTP Client | grpms_backend_* | GRP-MultiSeller → Backend | Requisições | includes/api/grpms-backend-client.php |

**Nível de acoplamento:** Médio (GRP-MultiSeller estende e integra). Linhas: Hooks VERIFICADO (frontend/grpms-main.php:1298, :1310, :1320); Shortcodes `[grp_*]` VERIFICADO em parte (grpms-dashboard.php:13-39; `[grp_dashboard]` não registrado); AJAX `wp_ajax_*` VERIFICADO (muitos módulos, ex.: grpms-products.php:2184, não só includes/admin/access/classes/); REST VERIFICADO (grpms-rest-api.php:908); HTTP Client VERIFICADO (grpms-backend-client.php:27)

---

### 4.4. GRP-MultiSeller ↔ Backend FastAPI

| Tipo | Mechanismo | Direção | Dados | Fontes |
|------|------------|---------|-------|--------|
| HTTP | POST/GET | Bidirecional | JSON | includes/api/grpms-backend-client.php |
| Autenticação | X-GRPMS-WP-Secret | GRP-MultiSeller → Backend | Header | fastapi_app/auth/wordpress_bridge.py |
| Tokens | JWT | Backend → GRP-MultiSeller | Token | fastapi_app/routers/token_router.py |

**Nível de acoplamento:** Baixo (HTTP REST com autenticação): avaliação do arquiteto, NAO VERIFICADO. Linhas: HTTP POST/GET VERIFICADO (método padrão POST em grpms-backend-client.php:53; chamadores de validate-affiliate usam GET: grpms-checkout.php:1287, grpms-guest-shipping.php:218); autenticação por header VERIFICADO (wordpress_bridge.py:10); tokens VERIFICADO (token_router.py:18)

**Rotas permitidas (VERIFICADO grpms-backend-client.php:33-35; `/api/checkout/save-commissions` sem rota no backend, NAO VERIFICADO):**
- `/api/token/from-wp`
- `/api/easypost/calculate-shipping`
- `/api/easypost/validate-selection`
- `/api/checkout/validate-affiliate`
- `/api/checkout/save-order`
- `/api/checkout/save-commissions`

**Fontes:**
- includes/api/grpms-backend-client.php:33-35

---

### 4.5. Backend FastAPI ↔ Banco de Dados

| Tipo | Mechanismo | Direção | Dados | Fontes |
|------|------------|---------|-------|--------|
| SQL | Queries | Backend → MySQL | Tabelas wp_* | fastapi_app/utils/database.py |
| Driver (não ORM) | pymysql | Backend → MySQL | Connection | fastapi_app/utils/database.py |

**Nível de acoplamento:** Alto (acesso direto ao BD WordPress): VERIFICADO (token_router.py:24, checkout_service.py:9, backend/fastapi_app/services/checkout/order_service.py:50)

---

### 4.6. Backend FastAPI ↔ Serviços Externos

| Serviço | Mechanismo | Direção | Dados | Fontes |
|---------|------------|---------|-------|--------|
| EasyPost | API REST (SDK `easypost`) | Backend → EasyPost | Cotação de frete | VERIFICADO easypost_service.py:10, :353; rotas em easypost_router.py:141 (chamada de rede real: NAO VERIFICADO) |
| Stripe (provedor segundo o PO; plugin NAO VERIFICADO) | Plugin do WooCommerce (mecanismo NAO VERIFICADO) | WooCommerce → Stripe | Pagamentos | Plugin e configuração fora dos repositórios; WooPayments citado só em documentação (README-FE.md:15); nenhuma chamada nos repositórios |
| PayPal (NAO VERIFICADO) | API REST | WordPress → PayPal | Pagamentos | WC PayPal plugin (README-FE.md:16, documentação); nenhuma chamada nos repositórios |
| Frankfurter | API REST | WordPress → Frankfurter | Taxas de câmbio | VERIFICADO frontend/grpms-main.php:1145-1163 |
| floatrates | API REST | WordPress → floatrates | Taxas de câmbio | VERIFICADO frontend/grpms-main.php:1175-1176 |

**Nível de acoplamento:** Baixo (API REST com autenticação): NAO VERIFICADO; observação: as chamadas a Frankfurter/floatrates não enviam credenciais no código (ver DATA-FLOW 2.5). A seção é intitulada "Backend ↔ Serviços Externos", mas Stripe, PayPal, Frankfurter e floatrates são acionados pelo WordPress, não pelo backend

---

## 5. Contratos de Integração

### 5.1. Contrato: WordPress → Backend (HTTP)

**Requisitos:**
- Header `X-GRPMS-WP-Secret` é enviado por `grpms_backend_http()` somente para as seis rotas permitidas (VERIFICADO grpms-backend-client.php:33-35, header em :46-48); chamadas de dados dos painéis usam JWT Bearer (VERIFICADO grpms-dashboard.php:152, grpms-gallery.php:177); rotas de rastreio EasyPost usam identidade de vendedor assinada por HMAC (VERIFICADO `X-GRPMS-Vendor-Id/Ts/Sig`, wordpress_bridge.py:18-35)
- Apenas rotas na whitelist são permitidas pelo cliente PHP (VERIFICADO grpms-backend-client.php:33-39); o backend em si não impõe essa lista
- Respostas são em JSON (VERIFICADO: `grpms_backend_request` exige corpo JSON em array, grpms-backend-client.php:58-70; `/save-order` retorna 204 sem corpo, backend/fastapi_app/routers/checkout/order_router.py:11)
- Erros retornam HTTP status codes apropriados (VERIFICADO em parte: 401/503 wordpress_bridge.py:13-15, 403 token_router.py:27, 409/500 backend/fastapi_app/routers/checkout/order_router.py:15-18; completude NAO VERIFICADA)

**Exemplo de requisição:**
```http
POST /api/token/from-wp HTTP/1.1
Host: <host-interno-do-backend>
X-GRPMS-WP-Secret: <segredo>
Content-Type: application/json

{
  "user_id": 123
}
```

**Exemplo de resposta:**
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Fontes:**
- VERIFICADO fastapi_app/auth/wordpress_bridge.py:10-15
- VERIFICADO includes/api/grpms-backend-client.php:1-110 (exemplo de requisição: host e corpo ilustrativos; token truncado de exemplo, não é um valor real)

---

### 5.2. Contrato: Backend → Banco de Dados (SQL)

**Requisitos:**
- Conexão via pymysql (MySQL, padrão) ou psycopg2 (PostgreSQL, caminho existente mas o esquema/SQL é específico de WordPress/MySQL; suporte efetivo NAO VERIFICADO)
- Credenciais via variáveis de ambiente
- Queries seguem schema do WordPress (tabelas wp_*)
- Conexão deve ser fechada após uso

**Exemplo de query:**
```sql
SELECT m.meta_value 
FROM wp_usermeta m 
JOIN wp_users u ON u.ID=m.user_id 
WHERE m.user_id=%s AND m.meta_key='wp_capabilities'
```

**Fontes:**
- VERIFICADO fastapi_app/utils/database.py:8-30
- VERIFICADO fastapi_app/routers/token_router.py:24

---

## 6. UNVERIFIED Items

- UNVERIFIED: Plugin, versão e configuração da integração entre WooCommerce e Stripe (WooPayments é só citação documental)
- UNVERIFIED: Detalhes completos da integração entre WooCommerce e PayPal
- UNVERIFIED: Estado de habilitação do webhook WooCommerce (código: padrão desligado, `GRPMS_AUTO_COMPLETE`) em qualquer ambiente
- UNVERIFIED: Detalhes da configuração do Nginx para roteamento de requisições
- UNVERIFIED: Como o WordPress lida com sessões e cookies em produção
- UNVERIFIED: Detalhes da configuração do MySQL (timeout, connection pool, etc.)
