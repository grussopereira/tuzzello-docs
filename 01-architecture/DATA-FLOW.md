> **STATUS DO DOCUMENTO: PARCIAL** (revisado em 2026-11-03 após validação ARCH-VALIDATE-01 / ARCH-FIX-01). Citações de linha corrigidas contra o HEAD de frontend e backend; o modelo de implantação descrito é o evidenciado pelas docs de implantação (VMs, não contêineres). Itens não confirmáveis no código estão marcados **NAO VERIFICADO**: Nginx, bloqueio público de rotas, status de produção de qualquer componente, Stripe/WooPayments, PayPal, Podman, YITH Affiliates, topologia/IPs, cron (apenas em README-FE.md), instalação do MU-Plugin de mídia. Flags com padrão desligado (`GRPMS_COMMISSION_WATERFALL`, `EASYPOST_BUY_LIVE`, `GRPMS_AUTO_COMPLETE`, `GRPMS_WC_SEND_ENABLED`) estão descritas como tal.
> **ARCH-FIX-04 (2026-11-03) — Stripe e systemd:** (a) Stripe (ARCH-FIX-05): segundo o Product Owner (informação do proprietário, aceita como declarada), o Stripe é o provedor de pagamento da plataforma, usado por meio de um plugin do WooCommerce. O mecanismo dentro do WordPress (qual plugin, versão e configuração) é **NAO VERIFICADO**: não está nestes repositórios. Nenhuma chamada à API Stripe, PaymentIntent, Checkout Session, webhook Stripe ou verificação de assinatura existe em `frontend/` e `backend/fastapi_app` neste HEAD (apenas fixtures em frontend/tests/payout-approval-guard.php:367,395); o checkout guarda o identificador genérico de método de pagamento do WooCommerce e o backend guarda o `transaction_id` recebido do webhook de pedido do WooCommerce. O ponto de integração é o WooCommerce, não o backend. Referências de configuração fora do código (somente nomes de arquivo e linhas; nenhum conteúdo aberto ou registrado): backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (drop-in `stripe.conf`) e :24 (arquivo `backend/infrastructure/secrets/stripe.env`, referência de arquivo apenas, ausente neste workspace); frontend/docs/REGISTRO-ERROS-CORRECOES.md:579. (b) systemd: não consta em CLOUD-DEPLOYMENT.md (frontend e backend); consta em backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (alegação do repositório lab, conflitante com a decisão do PO `grpms-backend`: serviço de usuário `tuzzello-backend`, **NAO VERIFICADO**) e frontend/docs/REGISTRO-ERROS-CORRECOES.md:579; nome de serviço: decisão do PO = `grpms-backend`; a citação `tuzzello-backend` (PROCESSO-AUTOMATICO-CONCLUSAO.md:22) é alegação do repositório conflitante, NAO VERIFICADO; código/implantação: **NAO VERIFICADO**. (c) Citações de arquivo usam caminho completo a partir da raiz do repositório (`frontend/…`, `backend/…`).
> **ARCH-NOTE (2026-11-03) — Nome do serviço (conflito resolvido):** o Product Owner decidiu `grpms-backend` (2026-11-03, decisão vinculante). O serviço de usuário systemd do LAB foi renomeado de `tuzzello-backend` para `grpms-backend` em 2026-11-03 (verificado somente leitura pelo relatório do engenheiro `/home/gilberto/tuzzello-specs/eng-rename-report.md`). A linha citada do documento do backend (`backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22`) foi atualizada no commit e75597f do repositório backend. Demais afirmações e o status **PARCIAL** do documento permanecem inalterados.

> **ARCH-FIX-02 (2026-11-03):** releitura contra o código de frontend e backend concluída; linhas corrigidas onde divergiam e toda afirmação não confirmável marcada **NAO VERIFICADO** no próprio texto. O documento permanece PARCIAL: nenhuma afirmação sobre plugins externos, produção ou topologia foi confirmada.

> **ARCH-FIX-03 (2026-11-03):** notas de seção convertidas em marcadores por afirmação. Convenção: **VERIFICADO (arquivo:linha)** = lido no código de `frontend/` ou `backend/` neste HEAD; **NAO VERIFICADO** = não confirmado por código (plugin externo, produção, topologia, documentação apenas). O documento é VERIFICADO somente se toda afirmação for confirmada; como restam afirmações NAO VERIFICADAS (Stripe/PayPal, MySQL em produção, webhook habilitado, topologia), o status permanece **PARCIAL**.

# Fluxos de Dados do Sistema Tuzzello

> **Propósito:** Documentar os fluxos de dados entre frontend, backend, banco de dados e serviços externos, incluindo boundaries de confiança. **Data:** 2026-11-03. Descreve o código no HEAD dos repositórios.

---

## 1. Visão Geral dos Fluxos

O sistema Tuzzello possui fluxos de dados complexos que cruzam **trust boundaries** entre:
1. **Navegador do Cliente** (inseguro) ↔ **WordPress** (parcialmente confiável)
2. **WordPress** (parcialmente confiável) ↔ **Backend FastAPI** (confiável)
3. **Backend FastAPI** (confiável) ↔ **Banco de Dados** (confiável)
4. **Backend FastAPI** (confiável) ↔ **Serviços Externos** (não confiáveis)

**Níveis de Confiança:**
- 🟢 **Confiável (Trusted):** Backend FastAPI, Banco de Dados
- 🟡 **Parcialmente Confiável:** WordPress (depende do contexto de execução)
- 🔴 **Não Confiável (Untrusted):** Navegador do Cliente, Serviços Externos

> Nota: os níveis de confiança desta seção são classificação do arquiteto, não afirmação verificável no código (NAO VERIFICADO).

---

## 2. Fluxos Principais

### 2.1. Fluxo de Autenticação e Geração de Token

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│   Navegador  │────▶│  WordPress   │────▶│  FastAPI Backend  │────▶│  BD (MySQL: NAO VERIFICADO) │
│   (Cliente)  │     │   (PHP)      │     │   (Python)        │     │              │
└─────────────┘     └─────────────┘     └─────────────────┘     └─────────────┘
       │                    │                      │                      │
       │  Cookies WP        │  Session WP          │  X-GRPMS-WP-Secret   │
       │  (JWT: ❌)          │  (Autenticado: ✅)    │  (Valido: ✅)        │
       │                    │                      │                      │
       └─────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                              Token JWT
                                    ▼
                       Retorna para Navegador
```

**Etapas:**
1. Usuário faz login no WordPress (🟡 Parcialmente Confiável)
2. WordPress valida credenciais e cria sessão (🟡→🟢 via cookies seguros; flags dos cookies: NAO VERIFICADO)
3. Frontend solicita token JWT para dashboard (VERIFICADO: `grpms_current_user_backend_token` grpms-backend-client.php:74-86 chama `/api/token/from-wp`; handler AJAX `grpms_refresh_dashboard_token` :88-110 exige login, nonce `grpms_dashboard_token`, role wcfm_vendor ou yith_affiliate e `GRPMS_Access_Validator::check`)
4. WordPress chama `/api/token/from-wp` no backend (VERIFICADO token_router.py:18-42; `/api/token` e `/api/token/login` retornam 410, token_router.py:13-16)
   - **Trust Boundary:** WordPress → Backend via `X-GRPMS-WP-Secret` (dependência `require_wordpress`, token_router.py:18; header definido em wordpress_bridge.py:10)
   - Backend valida segredo: `secrets.compare_digest()` (wordpress_bridge.py:14; segredo < 32 caracteres → 503, :12-13)
5. Backend consulta `wp_usermeta` para `wp_capabilities` (token_router.py:24)
   - **Trust Boundary:** Backend → BD (ambos 🟢 Confiáveis; classificação de confiança é avaliação do arquiteto, NAO VERIFICADO no código)
6. Backend parseia capabilities via `phpserialize.loads()` (token_router.py:29; verificado)
7. Backend gera token JWT com `create_token()` (token_router.py:35)
8. Backend retorna token JWT para WordPress
9. WordPress retorna token JWT para navegador

**Dados transmitidos:**
- WordPress → Backend: `{user_id: int}` (token_router.py:10-11)
- Backend → WordPress: `{access_token: string}` (token_router.py:35)

**Trust Boundary Crossed:**
- WordPress ↔ Backend: Autenticação via segredo compartilhado
- Backend ↔ BD: acesso direto (sem trust boundary; tipo de BD NAO VERIFICADO; seleção de engine em database.py:8-30)

---

### 2.2. Fluxo de Validação de Afiliado

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐
│   Navegador  │────▶│  WordPress   │────▶│  FastAPI Backend  │
│   (Visitante)│     │   (PHP)      │     │   (Python)        │
└─────────────┘     └─────────────┘     └─────────────────┘
       │                    │                      │
       │  Hash Afiliado      │  grpms_backend_http  │                      │
       │  (Product ID)       │  (Path: /api/checkout │                      │
       │                    │   /validate-affiliate)│                      │
       │                    │                      │
       │                    │  X-GRPMS-WP-Secret    │
       │                    │  (Validado)           │
       └────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                     Valida hash na base de dados
                                    ▼
                              Retorno: valid/invalid
```

**Etapas:**
1. Visitante clica em link de afiliado. VERIFICADO: o código lê os parâmetros de URL `hash` e `add-to-cart` (grpms-checkout.php:1283-1285); o exemplo anterior `?affiliate_hash=...&product_id=...` era ilustrativo e não corresponde ao código. A origem/geração do link (`generate_affiliate_hash`, grpms-affiliation.php:75) é VERIFICADA como função; o formato final da URL emitida: NAO VERIFICADO
2. WordPress captura hash e product_id. VERIFICADO: há dois chamadores de `/api/checkout/validate-affiliate` no frontend: (a) `grpms_validate_affiliate_via_api`, hook `template_redirect` (grpms-checkout.php:1281-1282), que lê `$_GET['hash']` e `$_GET['add-to-cart']` (:1283-1285) e chama o backend em **grpms-checkout.php:1287**; (b) revalidação do carrinho em `woocommerce_after_checkout_validation` prioridade 998 (grpms-guest-shipping.php:208), que chama o backend em **grpms-guest-shipping.php:218** com `hash` do snapshot de sessão e `product_id` do único item do carrinho (:213-219). Ambos usam método **GET** com parâmetros de query
3. WordPress chama `grpms_backend_http('/api/checkout/validate-affiliate', ['method' => 'GET', 'query' => [hash, product_id]])` (VERIFICADO: grpms-checkout.php:1287-1290 e grpms-guest-shipping.php:218-220; rota permitida em grpms-backend-client.php:33-35, header em :46-48; o método padrão do cliente é POST (:53), mas os dois chamadores sobrescrevem para GET, coerente com a rota `GET /validate-affiliate`, checkout_router.py:27)
   - **Trust Boundary:** WordPress → Backend via `X-GRPMS-WP-Secret`
4. Backend valida autenticação via `require_wordpress` (VERIFICADO: dependência do router, checkout_router.py:16)
5. Backend chama `validate_affiliate_link(hash, product_id)` (VERIFICADO: checkout_router.py:33; implementação em checkout_service.py:41-100 lida: consulta `grpms_affiliate_links` com `link_status = 'active'`, exige produto publicado/aprovado via `product_is_eligible` e afiliado com role `yith_affiliate` e KYC `approved` via `affiliate_is_eligible`, checkout_service.py:6-38)
6. Backend valida se produto está ativo e em estoque (VERIFICADO: erros `product_disabled`, `out_of_stock`, `invalid_link` → 403 em checkout_router.py:36-41; no serviço, `_stock_status = 'outofstock'` retorna `out_of_stock` e `grpms_prod_extra` retorna `product_disabled`, checkout_service.py:41-100)
7. Backend retorna status da validação (VERIFICADO: modelo de resposta checkout_router.py:21-25)

**Dados transmitidos:**
- Query params: `hash: string`, `product_id: int`
- Headers: `X-GRPMS-WP-Secret: string`

**Trust Boundary Crossed:**
- WordPress ↔ Backend: Autenticação via segredo compartilhado

**Resultados possíveis:**
- `200 OK`: `{product_id, quantity, discount, affiliate_id}` (checkout_router.py:21-25)
- `403 Forbidden`: `product_disabled`, `out_of_stock`, `invalid_link`

---

### 2.3. Fluxo de Checkout (Compra)

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│   Navegador  │     │  WordPress   │     │  FastAPI Backend  │     │ Stripe (via │
│   (Cliente)  │────▶│ (PHP)+WooComm│     │   (Python)        │     │ WooComm.:NV │
└─────────────┘     └─────────────┘     └─────────────────┘     └─────────────┘
       │                    │                      │                      │
       │  Dados Cart        │  Dados do Pedido      │  Validação           │  Processa   │
       │  (Sessão WC)       │                      │  Affiliado           │  Pagamento  │
       │                    │                      │                      │
       │                    │  X-GRPMS-WP-Secret    │                      │
       │                    │                      │                      │
       └─────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                     Salva pedido no WooCommerce
                                    ▼
                       Sincroniza com Backend
```

**Etapas:**
1. Cliente adiciona produtos ao carrinho (sessão WooCommerce)
2. Cliente começa checkout (página de checkout do WooCommerce)
3. **Validação de afiliado (opcional):**
   - Frontend obtém hash de afiliado (se presente)
   - WordPress valida hash via backend (fluxo 2.2)
4. Cliente preenche endereços de entrega e pagamento
5. **Cálculo de frete:**
   - WordPress obtém cotação de frete via backend (Seção 2.4)
6. Cliente confirma pedido
7. WordPress processa pagamento via gateway do WooCommerce (Stripe é o provedor segundo o PO, via plugin do WooCommerce; plugin, versão e configuração: NAO VERIFICADO; PayPal: NAO VERIFICADO; gateways são plugins externos. O backend não integra com Stripe: ele recebe o `transaction_id` pelo webhook de pedido do WooCommerce. Busca em `frontend/includes` e `backend/fastapi_app`: nenhuma chamada à API Stripe; sobre PayPal, o frontend apenas detecta ids de gateway `paypal`/`ppcp` para restringir moedas, grpms-checkout.php:1097-1125, e armazena o e-mail PayPal do vendedor para repasse, grpms-paypalset.php:20; nenhuma chamada à API PayPal. O backend lê chaves de metadado do WooPayments, `_wcpay_payment_transaction_id` e `_wcpay_transaction_fee`, em financial_fee_service.py:8-20, chamado de backend/fastapi_app/services/checkout/order_service.py:392)
8. **Salvar pedido:**
   - WordPress cria pedido no WooCommerce
   - WordPress chama `grpms_backend_request('/api/checkout/save-order', {...})` (VERIFICADO: chamador em includes/checkout/grpms-checkout.php:2029 usa `grpms_backend_http`, não `grpms_backend_request`; cliente grpms-backend-client.php:27-56; rota `POST /api/checkout/save-order`, backend/fastapi_app/routers/checkout/order_router.py:11-12)
   - Backend valida via `require_wordpress` (VERIFICADO: `dependencies=[Depends(require_wordpress)]` no router, backend/fastapi_app/routers/checkout/order_router.py:8)
   - Backend processa pedido e comissões (VERIFICADO: backend/fastapi_app/services/checkout/order_service.py `insert_order_into_main_tables`, :504; `save_commissions`, :156; reconciliação de itens contra `wp_woocommerce_order_items`, :42-80; erro de reconciliação → HTTP 409, backend/fastapi_app/routers/checkout/order_router.py:15-16)
9. **Salvar comissões:**
   - `/api/checkout/save-commissions` consta na lista de rotas permitidas (grpms-backend-client.php:35), mas NAO VERIFICADO: nenhum chamador no frontend e nenhuma rota com esse caminho foram encontrados no backend (VERIFICADO: `save_commissions` existe como função em backend/fastapi_app/services/checkout/order_service.py:156; a rota HTTP não existe)
   - Backend salva comissões no BD: NAO VERIFICADO como rota; comissões em cascata só com `GRPMS_COMMISSION_WATERFALL` (VERIFICADO padrão desligado: backend/fastapi_app/services/checkout/order_service.py:133-140; `compute_commissions` chamado em :199 somente se o flag estiver ativo)
10. WooCommerce exibe página de agradecimento (NAO VERIFICADO: comportamento do WooCommerce/CartFlows; o frontend adiciona apenas o shortcode `grpms_vendor_contact_info`, grpms-checkout-thankyou.php:7, e o guard de etiqueta, grpms-checkout-guard.php:57-73, VERIFICADOS)
11. **Sincronização de pedidos:** Pedidos sincronizam após pagamento (apenas frontend docs/CLOUD-DEPLOYMENT.md:17; NAO VERIFICADO no código)

**Dados transmitidos no save-order (exemplo ilustrativo, NAO VERIFICADO como payload real do chamador; campos reais em models.py `OrderPayload`; correspondência campo a campo NAO VERIFICADA; valores são fictícios):**
```json
{
  "order_id": 12345,
  "customer_id": 67890,
  "payment_method": "<id genérico do gateway WooCommerce>",
  "requires_shipping": true,
  "shipping": 15.50,
  "tax": 2.50,
  "shipping_tax": 0.0,
  "order_status": "processing",
  "affiliate_id": 111,
  "affiliate_hash": "XYZ123",
  "shipment_id": null,
  "rate_id": "rate_abc123",
  "items": [
    {
      "vendor_id": 10,
      "product_id": 123,
      "variation_id": null,
      "quantity": 2,
      "product_price": 50.00,
      "purchase_price": 50.00,
      "item_id": 1,
      "item_type": "line_item",
      "item_total": 100.00
    }
  ],
  "billing": {...},
  "shipping_address": {...}
}
```

**Trust Boundary Crossed:**
- WordPress ↔ Backend: Autenticação via segredo compartilhado
- WordPress (plugin de pagamento do WooCommerce) ↔ Stripe/PayPal: mecanismo e credenciais NAO VERIFICADOS (plugins externos, fora destes repositórios); o backend não integra diretamente
- Backend ↔ MySQL: Acesso direto (🟢→🟢)

---

### 2.4. Fluxo de Cálculo de Frete (EasyPost)

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│   Navegador  │     │  WordPress   │     │  FastAPI Backend  │     │  EasyPost   │
│   (Cliente)  │────▶│   (PHP)      │────▶│   (Python)        │────▶│   (Frete)   │
└─────────────┘     └─────────────┘     └─────────────────┘     └─────────────┘
       │                    │                      │                      │
       │  Endereço          │  Dados de cotação    │  Requisição API      │  Retorna     │
       │  (Carrinho)        │  (Produtos, Peso)    │  (EasyPost API)      │  Cotações    │
       │                    │                      │                      │
       │                    │  X-GRPMS-WP-Secret    │  EASYPOST_TEST_KEY   │
       │                    │                      │  (ou LIVE_KEY)       │
       └─────────────────────────────────────────────────────────────────┘
```

**Etapas:**
1. Cliente insere endereço de entrega
2. WordPress coleta dados do carrinho (produtos, pesos, dimensões)
3. WordPress chama `grpms_backend_request('/api/easypost/calculate-shipping', {...})` (VERIFICADO: rota permitida em grpms-backend-client.php:33-34; chamador `grpms_backend_request` em grpms-guest-shipping.php:107, acionado pelas ações AJAX `grpms_shipping_quote`/`grpms_shipping_select`, :157-160; rota `POST /api/easypost/calculate-shipping`, easypost_router.py:141; a seleção de cotação é validada via `/api/easypost/validate-selection`, grpms-guest-shipping.php:141)
   - **Trust Boundary:** WordPress → Backend via `X-GRPMS-WP-Secret`
4. Backend valida autenticação via `require_wordpress` (VERIFICADO: dependência do router, easypost_router.py:16)
5. Backend chama EasyPost API com dados do endereço e produto (VERIFICADO: `EasyPostService.calculate_shipping`, easypost_service.py:353, usa o SDK `easypost.EasyPostClient`, importado em :10; a chamada de rede à EasyPost em si só ocorre em runtime: NAO VERIFICADO)
   - **Trust Boundary:** Backend → EasyPost via API Key (🟢→🔴)
   - Modo: `EASYPOST_MODE=test` ou `live` (VERIFICADO backend/.env.example:10)
   - Chave: `EASYPOST_TEST_KEY` ou `EASYPOST_LIVE_KEY` (VERIFICADO backend/.env.example:11-12)
6. EasyPost retorna cotações disponíveis
7. Backend valida e formata resposta
8. WordPress exibe opções de frete para cliente

**Dados transmitidos:**
- Endereço de origem (vendedor)
- Endereço de destino (cliente)
- Dimensões e peso do produto
- Tipo de serviço desejado

**Trust Boundary Crossed:**
- WordPress ↔ Backend: Autenticação via segredo compartilhado
- Backend ↔ EasyPost: Autenticação via API Key (serviço externo não confiável)

---

### 2.5. Fluxo de Consulta de Taxas de Câmbio

```
┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│   WordPress   │     │  FastAPI Backend  │     │ Frankfurter  │
│   (PHP)      │     │   (Python)        │     │   (Câmbio)   │
└─────────────┘     └─────────────────┘     └─────────────┘
       │                      │                      │
       │  grpms_get_exchange  │  Solicitação HTTP     │  API REST     │
       │  _rate_to_usd()      │  (Providers)          │              │
       │                      │                      │
       └─────────────────────────────────────────────────────────────┘
```

**Etapas:**
1. WordPress detecta necessidade de conversão de moeda (VERIFICADO frontend/grpms-main.php:1126; providers :1145-1176; cache :1221; chamadores: `grpms_convert_amount_to_usd`, frontend/grpms-main.php:1255, e grpms-checkout.php:1439-1440)
2. WordPress chama `grpms_get_exchange_rate_to_usd(from_currency)`
3. Verifica cache: `get_transient('grpms_fx_' . strtolower($from_currency) . '_usd_' . gmdate('Ymd'))` (VERIFICADO frontend/grpms-main.php:1136; transient)
4. Se cache válido, retorna taxa cached (posição exata: NAO VERIFICADO)
5. Se cache inválido, tenta providers em ordem:
   - **frankfurter.dev/v2** (frontend/grpms-main.php:1145-1146)
     - URL: `https://api.frankfurter.dev/v2/rate/{from_currency}/USD`
   - **frankfurter.dev/v1** (frontend/grpms-main.php:1157-1163)
     - URL: `https://api.frankfurter.dev/v1/latest?base={from_currency}&symbols=USD`
   - **floatrates.com** (frontend/grpms-main.php:1175-1176)
     - URL: `https://www.floatrates.com/daily/{from_currency}.json`
6. Armazena em cache por 24h (DAY_IN_SECONDS, :1221) ou 1h (HOUR_IN_SECONDS, :1240) para fallback; opção `grpms_fx_last_valid_*` (VERIFICADO: nome em frontend/grpms-main.php:1142, gravada com `update_option` e lida como fallback em ~:1225-1242)
7. Retorna taxa e data

**Dados transmitidos:**
- `from_currency`: Código da moeda (EUR, BRL, ZAR, XOF)
- Retorno: `{rate: float, date: string, source: string}` (formato exato: NAO VERIFICADO; `source` confirmado em :1145)

**Trust Boundary Crossed:**
- WordPress ↔ Frankfurter/floatrates: Requisição HTTP para API pública (🟡→🔴)
  - sem autenticação necessária

---

### 2.6. Fluxo de Sincronização de Pedidos (Webhook WooCommerce) — **padrão desligado**

> **Correção:** a versão anterior afirmava que webhooks estão desativados e descrevia um fluxo `order.created` com sincronização incondicional. No código: o endpoint assinado existe, mas só responde com `GRPMS_AUTO_COMPLETE` ativo (senão 404); só aceita o tópico `order.updated`; exige assinatura HMAC (`GRPMS_WC_WEBHOOK_SECRET`). O estado real em qualquer ambiente é NAO VERIFICADO.

```
┌─────────────────┐     ┌─────────────┐     ┌─────────────────┐
│   WooCommerce    │     │  FastAPI     │     │   WordPress BD   │
│   (Webhook)      │────▶│  Backend     │────▶│   (wp_*)        │
└─────────────────┘     └─────────────┘     └─────────────────┘
       │                      │                      │
       │  Evento de Pedido     │  Processamento        │  Atualização    │
       │  (order.updated)      │                      │  de Status      │
       │                      │                      │
       └─────────────────────────────────────────────────────────────┘
```

**Etapas:**
1. (Se configurado no WooCommerce — NAO VERIFICADO) WooCommerce envia webhook de tópico `order.updated`
2. Backend recebe em `POST /webhooks/woocommerce/order` (VERIFICADO routers/webhooks/woocommerce_order.py; registrado em main.py:24). Se `GRPMS_AUTO_COMPLETE` não estiver ativo → 404; corpo > 1 MiB → 413
3. `wc_webhook.verify` valida a assinatura (VERIFICADO: `x-wc-webhook-signature`, HMAC-SHA256 base64, `GRPMS_WC_WEBHOOK_SECRET`; wc_webhook.py:52-70); outros tópicos são ignorados (:78)
4. `process_event` grava a entrega (idempotência por `delivery_id`) e opera no BD (wc_webhook.py:126-)
5. `wc_outbox_worker.py` (lote único, sem scheduler instalado) só escreve de volta no WooCommerce com `GRPMS_WC_SEND_ENABLED`, `GRPMS_AUTO_COMPLETE` e `GRPMS_ENV=lab`; `auto_complete.py` é "unwired, default-off"
   - O detalhe de efeitos no BD além do registro de entrega: NAO VERIFICADO nesta revisão

**Fluxo alternativo (frontend docs/CLOUD-DEPLOYMENT.md:28-29; easypost_router.py:85, :190-192):**
- Compra de etiquetas e webhook EasyPost retornam 503 (esse comportamento NÃO se aplica ao endpoint WooCommerce acima)
- Homologar checkout primeiro, depois implementar fulfillment

---

## 3. Fluxos Internos do WordPress

### 3.1. Fluxo de Bloqueio de Acesso Móvel

```
┌─────────────┐     ┌─────────────┐
│   Navegador  │────▶│  WordPress   │
│   (Mobile)   │     │   (PHP)      │
└─────────────┘     └─────────────┘
       │                    │
       │  User-Agent        │
       │                    │
       └────────────────────┬─────────────────────┘
                            │
               ┌────────────────┴────────────────┐
               ▼                                 ▼
    ┌─────────────────┐              ┌─────────────────┐
    │ /mydash/*       │              │ /app/*          │
    │                 │              │                 │
    │ Redireciona     │              │ Redireciona     │
    │ para /no-mobile │              │ para /no-mobile │
    │ -access         │              │ -access         │
    └─────────────────┘              └─────────────────┘
```

**Implementação:**
- Hook: `login_init` (frontend/grpms-main.php:79-81)
- Hook: `template_redirect` prioridade 5, `grpms_block_mobile_in_mydash` (frontend/grpms-main.php:113-127); `grpms_public_pages_gate` prioridade 9 (:727, :805); hook adicional `bloquear_mobile_em_mydash` (:1357-1362)
- Função: `grpms_is_mobile_user_agent()` (frontend/grpms-main.php:42)
- Função: `grpms_request_targets_pwa()` (frontend/grpms-main.php:55)
- Redireciona para: `/no-mobile-access` (frontend/grpms-main.php:81, 122)

**Paths bloqueados (VERIFICADO nas linhas citadas):**
- `/mydash` e sub-rotas: bloqueio móvel em frontend/grpms-main.php:119-120
- `/app` e sub-rotas: `grpms_is_app_path` (frontend/grpms-main.php:48-52)
- Qualquer path não na whitelist: redireciona para `/mydash/` no gate público (frontend/grpms-main.php:786-795; ver 3.3, nota sobre a semântica)

**Paths permitidos (whitelist):**
- `/`, `/app`, `/checkout`, `/cart`, `/shipping-calc`, `/checkout-timeout`, `/step/store-checkout-thank-you-02`, `/es`, `/pt`, `/my-account`, `/sold-out`, `/sorryfor`, `/url-invalid`, `/t-help`, `/tuzzreg`, `/south-africa`, `/cookie-policy-*`, `/general-terms-*`, `/no-mobile-access`, `/privacy-policy-*`, `/purchase-terms-*`, `/refund-policy-*`, `/notallowed` (frontend/grpms-main.php:745-775; lista completa lida: itens como `/refund-policy-*` e `/notallowed` NAO VERIFICADOS por não terem sido localizados nas linhas lidas)

**Trust Boundary:** Navegador → WordPress (User-Agent pode ser forjado; "proteção extra via cookie": NAO VERIFICADO, não localizado nas linhas lidas)

---

### 3.2. Fluxo de Forçar Produtos Virtuais

**Contexto:** Vendedores apenas podem cadastrar produtos virtuais (frontend/grpms-main.php:1264)

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Vendedor   │────▶│  WordPress   │────▶│  WooCommerce │
│   (Logado)   │     │   (PHP)      │     │   (Salvar    │
└─────────────┘     └─────────────┘     │   Produto)   │
                                  │     └─────────────┘
                                  │            │
                                  │            ▼
                                  │  Força _virtual = yes
                                  │
                                  ▼
                       Update post_meta
```

**Implementação:**
- Função: `grpms_should_force_vendor_virtual_products()` (VERIFICADO frontend/grpms-main.php:1264; hooks em :1298, :1310, :1320, :1321)
  - Verifica se usuário está logado
  - Verifica se tem role `wcfm_vendor`
  - Verifica se NÃO tem role `manage_woocommerce` (admin) — NAO VERIFICADO (corpo da função em :1264-1268 não reconferido)
- Hook: `save_post_product` prioridade 999 (frontend/grpms-main.php:1298)
- Função: `grpms_force_vendor_product_virtual_on_save()`
  - Exceções: AJAX (save_product_changes), autosave, revisions — NAO VERIFICADO (corpo de :1279 não reconferido)
  - Chama `grpms_force_product_virtual_meta()`
- Função: `grpms_force_product_virtual_meta()` (frontend/grpms-main.php:1270)
  - `update_post_meta($product_id, '_virtual', 'yes')`
- Filtro: `wcfm_product_manager_data_filter` (frontend/grpms-main.php:1300-1310)
  - Força `is_virtual = 'enable'` e `virtual = 'enable'` nos dados do WCFM
- Hook: `after_wcfm_products_manage_meta_save` (frontend/grpms-main.php:1320)
- Hook: `wcfm_products_manage_meta_save` (frontend/grpms-main.php:1321)
  - Chama `grpms_force_wcfm_product_virtual_after_save()` (frontend/grpms-main.php:1312)

---

### 3.3. Fluxo de Gatekeeper de Páginas Públicas

```
┌─────────────┐     ┌─────────────┐
│   Navegador  │────▶│  WordPress   │
│   (Visitante)│     │   (PHP)      │
└─────────────┘     └─────────────┘
                    │
                    ▼
         ┌───────────────────────┐
         │ Exceções Técnicas?    │
         │ (is_admin, REST, AJAX, │
         │  CRON, customize,     │
         │  embed)               │
         └───────────┬───────────┘
                     │
              ┌──────┴──────┐
              ▼             ▼
    ┌──────────────┐ ┌──────────────┐
    │    SIM       │ │    NÃO       │
    │  Permitir   │ │             │
    │  Acesso     │ │  Verificar   │
    └──────────────┘ │  Whitelist   │
                      └──────┬──────┘
                             │
              ┌──────────────┴──────────────┐
              │                              │
              ▼                              ▼
    ┌─────────────────┐            ┌─────────────────┐
    │   Path na       │            │   Redirecionar   │
    │   Whitelist?    │            │   para /mydash/  │
    │                │            │                  │
    └────────┬────────┘            └─────────────────┘
              │
              ▼
    ┌─────────────────┐
    │    SIM:         │
    │  Permitir      │
    │  Acesso        │
    └─────────────────┘
```

**Implementação:**
- Função: `grpms_public_pages_gate()` (frontend/grpms-main.php:727; add_action em :805)
- Hook: `template_redirect` com prioridade 9 (frontend/grpms-main.php:805)
- Exceções técnicas (frontend/grpms-main.php:728-734):
  - `is_admin()`
  - `REST_REQUEST`
  - `DOING_AJAX`
  - `DOING_CRON`
  - `is_customize_preview()`
  - `is_user_logged_in() && is_embed()`
- Whitelist de paths (frontend/grpms-main.php:745-775)
- Verificação de `/mydash/*`, `/app/*`, `/my-account` e passos CartFlows públicos (frontend/grpms-main.php:781-786)
- Redireciona para: `home_url('/mydash/')` (frontend/grpms-main.php:791)
- Nota: no diagrama acima, "SIM: Permitir" também cobre `/mydash`, `/app` e `/my-account`

---

### 3.4. Fluxo de Atribuição de Moeda do Produto

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Vendedor   │────▶│  WordPress   │────▶│   Produto   │
│   (Salvar)   │     │   (PHP)      │     │   (wp_posts)│
└─────────────┘     └─────────────┘     └─────────────┘
       │                    │
       │                    │ grpms_assign_product_currency_from_vendor()
       │                    │
       └─────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                          Meta: _grpms_product_currency
```

**Implementação:**
- Função: `grpms_assign_product_currency_from_vendor()` (frontend/grpms-main.php:1089-1105)
  - Chamada em: `save_post_product` hook (Prioridade 999) — NAO VERIFICADO (ponto de chamada não localizado nesta revisão)
  - Não sobrescreve (a menos que `$overwrite`) se já existir meta `_grpms_product_currency`
  - Obtém vendor_id do post_author ou current_user
  - Obtém moeda preferida via `grpms_get_user_preferred_currency()`
  - Salva: `update_post_meta($product_id, '_grpms_product_currency', $currency)`
- Função: `grpms_get_product_currency()` (frontend/grpms-main.php:1110-1123)
  - Busca meta `_grpms_product_currency`
  - Fallback: moeda do vendor (post_author)
  - Fallback: moeda do usuário atual

---

## 4. Fluxos de Dados do Backend

### 4.1. Fluxo de Processamento de Pedido

```
┌─────────────────┐     ┌─────────────┐     ┌─────────────────┐
│   WooCommerce    │     │  Backend     │     │   Database      │
│   (Webhook/Request)│────▶│  FastAPI     │────▶│   MySQL         │
└─────────────────┘     └─────────────┘     └─────────────────┘
       │                      │                      │
       │  OrderPayload        │  order_service       │  tabelas (NAO VERIFICADO)
       │                      │  .reconciliation()   │  
       │                      │                      │
       └─────────────────────────────────────────────────────────────┘
```

**Etapas (backend/fastapi_app/services/checkout/order_service.py):**
1. Recebe `OrderPayload` (VERIFICADO models.py:51; backend/fastapi_app/routers/checkout/order_router.py:12)
2. Valida itens com `canonical_items()` (VERIFICADO backend/fastapi_app/services/checkout/order_service.py:42-84: lê itens em `wp_woocommerce_order_items`/`itemmeta`)
   - Compara com dados no BD WordPress
   - Verifica: product_id, quantity, total
3. Se divergência, `canonical_items` registra via `reconciliation()` e lança `RuntimeError` (backend/fastapi_app/services/checkout/order_service.py:80-84); `OrderReconciliationError` é a classe em :28 e o router a mapeia para HTTP 409 (backend/fastapi_app/routers/checkout/order_router.py:15-16); onde exatamente `RuntimeError` é convertida em `OrderReconciliationError`: NAO VERIFICADO
4. Processa comissões (`save_commissions`, :156); "processa pagamento": NAO VERIFICADO (nenhum código de cobrança no backend foi localizado)
5. Atualiza status do pedido (`log_order_status_change`, :466; NAO VERIFICADO se é este o passo)

**Dados validados:**
- `order_id`: ID do pedido no WooCommerce
- `customer_id`: ID do cliente
- `payment_method`: Método de pagamento
- `items`: Lista de OrderItem (product_id, quantity, price)
- `billing`: Endereço de cobrança
- `shipping_address`: Endereço de entrega

---

## 5. Mapeamento de Trust Boundaries

| Fluxo | Origem | Destino | Trust Boundary | Mecanismo de Segurança | Nível de Risco |
|-------|--------|---------|----------------|------------------------|---------------|
| Token JWT | WordPress | Backend | ✅ Sim | `X-GRPMS-WP-Secret` | Baixo |
| Validação Afiliado | WordPress | Backend | ✅ Sim | `X-GRPMS-WP-Secret` | Baixo |
| Save Order | WordPress | Backend | ✅ Sim | `X-GRPMS-WP-Secret` | Baixo |
| Save Commissions (rota NAO VERIFICADA) | WordPress | Backend | ✅ Sim | `X-GRPMS-WP-Secret` | Baixo |
| EasyPost Shipping | Backend | EasyPost | ✅ Sim | API Key | Médio |
| Stripe Payment (provedor segundo o PO; plugin WooCommerce NAO VERIFICADO) | WooCommerce (WordPress) | Stripe | ✅ Sim | NAO VERIFICADO | Médio |
| PayPal Payment (NAO VERIFICADO) | WordPress | PayPal | ✅ Sim | API Key | Médio |
| Taxa de Câmbio | WordPress | Frankfurter | ✅ Sim | Nenhum (API pública) | Baixo |
| Backend → MySQL | Backend | MySQL | ❌ Não | Conexão direta | Baixo |
| WordPress → MySQL | WordPress | MySQL | ❌ Não | Conexão WordPress | Baixo |
| Navegador → WordPress | Navegador | WordPress | ✅ Sim | Cookies WordPress | Médio |

**Observações:**
- Todas as chamadas de WordPress para Backend exigem header `X-GRPMS-WP-Secret` (VERIFICADO para token, checkout/validate-affiliate, easypost e backend/fastapi_app/routers/checkout/order_router.py:8; rotas de rastreio também exigem identidade do vendedor via `require_vendor_identity`, easypost_router.py:89)
- Backend valida segredo via `secrets.compare_digest()` (comparação constante-time)
- Apenas rotas pré-aprovadas podem ser chamadas pelo cliente PHP (VERIFICADO grpms-backend-client.php:33-35)
- Backend acessa o BD diretamente com credenciais configuradas em .env (tipo de BD e uso exato de .env: NAO VERIFICADO)

---

## 6. UNVERIFIED Items

- UNVERIFIED: Efeitos completos do webhook WooCommerce no BD e se está habilitado em algum ambiente (seção 2.6 resume o que o código mostra)
- UNVERIFIED: Plugin, versão e configuração da integração Stripe no WooCommerce (WooPayments é citado só em documentação, README-FE.md:15; plugin externo)
- UNVERIFIED: Detalhes da integração com PayPal via WooCommerce PayPal Payments (plugin externo)
- UNVERIFIED: Fluxo completo de fulfillment (envio de produtos físicos)
- UNVERIFIED: Fluxo de reembolso (refund) completo
- UNVERIFIED: Sincronização de estoque entre vendedores
