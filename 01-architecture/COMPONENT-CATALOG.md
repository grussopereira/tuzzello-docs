> **STATUS DO DOCUMENTO: PARCIAL** (revisado em 2026-11-03 após validação ARCH-VALIDATE-01 / ARCH-FIX-01). Citações de linha corrigidas contra o HEAD de frontend e backend; o modelo de implantação descrito é o evidenciado pelas docs de implantação (VMs, não contêineres). Itens não confirmáveis no código estão marcados **NAO VERIFICADO**: Nginx, bloqueio público de rotas, status de produção de qualquer componente, Stripe/WooPayments, PayPal, Podman, YITH Affiliates, topologia/IPs, cron (apenas em README-FE.md), instalação do MU-Plugin de mídia. Flags com padrão desligado (`GRPMS_COMMISSION_WATERFALL`, `EASYPOST_BUY_LIVE`, `GRPMS_AUTO_COMPLETE`, `GRPMS_WC_SEND_ENABLED`) estão descritas como tal.
> **ARCH-FIX-04 (2026-11-03) — Stripe e systemd:** (a) Stripe (ARCH-FIX-05): segundo o Product Owner (informação do proprietário, aceita como declarada), o Stripe é o provedor de pagamento da plataforma, usado por meio de um plugin do WooCommerce. O mecanismo dentro do WordPress (qual plugin, versão e configuração) é **NAO VERIFICADO**: não está nestes repositórios. Nenhuma chamada à API Stripe, PaymentIntent, Checkout Session, webhook Stripe ou verificação de assinatura existe em `frontend/` e `backend/fastapi_app` neste HEAD (apenas fixtures em frontend/tests/payout-approval-guard.php:367,395); o checkout guarda o identificador genérico de método de pagamento do WooCommerce e o backend guarda o `transaction_id` recebido do webhook de pedido do WooCommerce. O ponto de integração é o WooCommerce, não o backend. Referências de configuração fora do código (somente nomes de arquivo e linhas; nenhum conteúdo aberto ou registrado): backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (drop-in `stripe.conf`) e :24 (arquivo `backend/infrastructure/secrets/stripe.env`, referência de arquivo apenas, ausente neste workspace); frontend/docs/REGISTRO-ERROS-CORRECOES.md:579. (b) systemd: não consta em CLOUD-DEPLOYMENT.md (frontend e backend); consta em backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (alegação do repositório lab, conflitante com a decisão do PO `grpms-backend`: serviço de usuário `tuzzello-backend`, **NAO VERIFICADO**) e frontend/docs/REGISTRO-ERROS-CORRECOES.md:579; nome de serviço: decisão do PO = `grpms-backend`; a citação `tuzzello-backend` (PROCESSO-AUTOMATICO-CONCLUSAO.md:22) é alegação do repositório conflitante, NAO VERIFICADO; código/implantação: **NAO VERIFICADO**. (c) Citações de arquivo usam caminho completo a partir da raiz do repositório (`frontend/…`, `backend/…`).
> **ARCH-NOTE (2026-11-03) — Nome do serviço (conflito resolvido):** o Product Owner decidiu `grpms-backend` (2026-11-03, decisão vinculante). O serviço de usuário systemd do LAB foi renomeado de `tuzzello-backend` para `grpms-backend` em 2026-11-03 (verificado somente leitura pelo relatório do engenheiro `/home/gilberto/tuzzello-specs/eng-rename-report.md`). A linha citada do documento do backend (`backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22`) foi atualizada no commit e75597f do repositório backend. Demais afirmações e o status **PARCIAL** do documento permanecem inalterados.

> **ARCH-FIX-02 (2026-11-03):** releitura contra o código de frontend e backend concluída; linhas corrigidas onde divergiam e toda afirmação não confirmável marcada **NAO VERIFICADO** no próprio texto. O documento permanece PARCIAL: nenhuma afirmação sobre plugins externos, produção ou topologia foi confirmada.

> **ARCH-FIX-03 (2026-11-03):** as notas de seção "inferido dos nomes de arquivo" foram substituídas por marcadores por afirmação após leitura de `frontend/includes/*` e `backend/fastapi_app/services/*`. Convenção: **VERIFICADO (arquivo:linha)** = confirmado no código deste HEAD; **NAO VERIFICADO** = não confirmado por código. Plugins externos (Stripe/WooPayments, PayPal, WCFM, YITH) são NAO VERIFICADOS salvo onde o código destes repositórios os referencia, e então a referência é citada. Correções de conteúdo feitas na leitura: `includes/grpms-commissions.php` NÃO calcula comissões (11 linhas, template que imprime `the_content()`); `includes/checkout/grpms-shipping-calculator.php` é um page template "Shipping Calculator" e não o template Add Product. O documento permanece **PARCIAL**.

**Catálogo de Componentes do Sistema Tuzzello**  
***Propósito:*** *Catálogo detalhado de todos os módulos do WordPress Plugin GRP-MultiSeller e serviços FastAPI com responsabilidades, entry points e paths.* ***Data:*** *2026-11-03. Descreve o código no HEAD dos repositórios.*  
  
---  
**Índice**  
1. [WordPress Plugin GRP-MultiSeller](#anchor-1 "#anchor-1")  
  - [1.1. Arquivo Principal](#anchor-2 "#anchor-2")  
  - [1.2. Módulos de API](#anchor-3 "#anchor-3")  
  - [1.3. Módulos de Acesso e Controle](#anchor-4 "#anchor-4")  
  - [1.4. Módulos de Afiliação](#anchor-5 "#anchor-5")  
  - [1.5. Módulos de Checkout](#anchor-6 "#anchor-6")  
  - [1.6. Módulos de Dashboard](#anchor-7 "#anchor-7")  
  - [1.7. Módulos Financeiros](#anchor-8 "#anchor-8")  
  - [1.8. Módulos de Galeria](#anchor-9 "#anchor-9")  
  - [1.9. Módulos Gerais](#anchor-10 "#anchor-10")  
  - [1.10. Módulos de Perfil](#anchor-11 "#anchor-11")  
  - [1.11. Módulos de Produtos](#anchor-12 "#anchor-12")  
  - [1.12. Módulos de Vendedor](#anchor-13 "#anchor-13")  
  - [1.13. Módulos de Treinamento](#anchor-14 "#anchor-14")  
  - [1.14. Módulos de PWA](#anchor-15 "#anchor-15")  
  - [1.15. Módulos de Plataforma](#anchor-16 "#anchor-16")  
  - [1.16. Módulos de KYC](#anchor-17 "#anchor-17")  
  - [1.17. Templates](#anchor-18 "#anchor-18")  
  - [1.18. Arquivos Estáticos](#anchor-19 "#anchor-19")  
2. [Backend FastAPI](#anchor-20 "#anchor-20")  
  - [2.1. Aplicação Principal](#anchor-21 "#anchor-21")  
  - [2.2. Módulos de Autenticação](#anchor-22 "#anchor-22")  
  - [2.3. Routers](#anchor-23 "#anchor-23")  
  - [2.4. Services](#anchor-24 "#anchor-24")  
  - [2.5. Models](#anchor-25 "#anchor-25")  
  - [2.6. Utilities](#anchor-26 "#anchor-26")  
  
---  
**1. WordPress Plugin GRP-MultiSeller**  
**1.1. Arquivo Principal**  

> **Cobertura (ARCH-FIX-03):** os módulos `includes/registry/grpms-registry.php`, `includes/admin/tax_liability`, `approve_orders` e `orders_history` são carregados em frontend/grpms-main.php:1480-1508 (VERIFICADO) e estão catalogados agora na seção 1.19. Os services backend `easypost_service`, `refund_*` etc. foram lidos e estão descritos na seção 2.4.1 com marcadores por afirmação.
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **GRP-MultiSeller Plugin** |  Plugin principal que registra roles, hooks e carrega módulos |  nenhuma função principal localizada; o arquivo executa no carregamento do plugin (VERIFICADO por leitura de frontend/grpms-main.php; ausência de função `grpms_main` confirmada por busca) |  `frontend/grpms-main.php` |  Versão 1.1.0 (VERIFICADO frontend/grpms-main.php:5) |  
  
   
**Responsabilidades:**  
- Registro da role `tuzzello_student` (VERIFICADO frontend/grpms-main.php:13-21; slug em :21)  
- Funções utilitárias: `grpms_normalize_path()`, `grpms_is_mobile_user_agent()`, `grpms_is_app_path()` (`grpms_normalize_path` :33, `grpms_is_mobile_user_agent` :42, `grpms_is_app_path` :48-52; corrigido)  
- Bloqueio de login em mobile para `/mydash/` (login_init :79-81; `grpms_block_mobile_in_mydash` :113-127; `bloquear_mobile_em_mydash` :1357-1362; corrigido)  
- Renderização de página "Desktop Access Required" (VERIFICADO `grpms_render_no_mobile_access_page` :136; fim da função e texto exato da página: NAO VERIFICADO)  
- Gatekeeper de páginas públicas (frontend/grpms-main.php:727; add_action em :805)  
- Bloqueio de páginas sem contexto (frontend/grpms-main.php:482-590): NAO VERIFICADO e a citação provavelmente é incorreta: as linhas 482-590 contêm CSS (`.status-card` etc.) da página de acesso móvel, não código comentado; nenhum bloqueio "sem contexto" foi localizado nesse intervalo. O gatekeeper real é `grpms_public_pages_gate` (:727, :805)  
- Funções de moeda: `grpms_supported_currency_codes()`, `grpms_normalize_currency_code()`, `grpms_currency_for_country()`, etc. (VERIFICADO: `grpms_supported_currency_codes` :807, `grpms_normalize_currency_code` :811, `grpms_currency_for_country` :817)  
- Funções de vendedor: `grpms_user_is_vendor()`, `grpms_get_seller_account_type()`, etc. (VERIFICADO: `grpms_user_is_vendor` :904, `grpms_get_seller_account_type` :935)  
- Forçar produtos virtuais para vendedores (VERIFICADO frontend/grpms-main.php:1264-1321)  
- Funções de taxa de câmbio: `grpms_get_exchange_rate_to_usd()`, `grpms_convert_amount_to_usd()` (VERIFICADO: `grpms_get_exchange_rate_to_usd` :1126; `grpms_convert_amount_to_usd` :1254; cache :1221)  
- Carregamento de módulos via `require_once` (VERIFICADO frontend/grpms-main.php:1480-1508)  
- Autoload de classes (VERIFICADO: `grpms_autoload` :1537; `spl_autoload_register` :1563)  
- Definição de constantes: `GRPMS_PLUGIN_DIR` (VERIFICADO frontend/grpms-main.php:1453)  
**Dependências:**  
- WordPress 6.0+ e WooCommerce 7.0+: requisitos declarados em README-FE.md:13-14 (documentação); versões NAO VERIFICADAS no código
- WCFM: o código depende dele de forma indireta: filtros/ações `wcfm_product_manager_data_filter` e `after_wcfm_products_manage_meta_save` (VERIFICADO frontend/grpms-main.php:1310, :1320) e a role `wcfm_vendor` (frontend/grpms-main.php:904 região e grpms-backend-client.php:95); existência, versão e comportamento do plugin WCFM: NAO VERIFICADO (plugin externo)  
  
---  
**1.2. Módulos de API**  
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Backend Client** |  Cliente HTTP para comunicação com backend FastAPI |  `grpms_backend_http()`, `grpms_backend_request()` |  `includes/api/grpms-backend-client.php` |  Segurança: Valida URLs, segredo compartilhado |  
| **REST API** |  API REST customizada do plugin |  `grpms_rest_namespace()` |  `includes/api/grpms-rest-api.php` |  Namespace: grpms/v1 |  
| **Guest Shipping** |  Lógica de frete para visitantes (não autenticados) (VERIFICADO: ações `wp_ajax_nopriv_grpms_shipping_quote`/`_select`, grpms-guest-shipping.php:157-160; chama `/api/easypost/calculate-shipping` :107 e `/api/easypost/validate-selection` :141; revalida afiliado :218) |  `grpms_guest_shipping_request` (VERIFICADO :56) |  `includes/api/grpms-guest-shipping.php` |  - |  
  
   
**Backend Client (grpms-backend-client.php):**  
- `grpms_validate_base_url()`: Validação de URLs (VERIFICADO :5)  
- `grpms_api_url()`: Construção de URLs da API (VERIFICADO :22)  
- `grpms_backend_http()`: Requisição HTTP, não genérica: restrita à lista de rotas (VERIFICADO linha 27-56; comentário "This is not a generic proxy" em :32)  
  - Valida configuração (GRPMS_API_INTERNAL_URL, GRPMS_WP_SHARED_SECRET)  
  - Verifica rota permitida (linhas 33-35)  
  - Adiciona header `X-GRPMS-WP-Secret` (linhas 46-48; enviado apenas por esta função, não em "todas as requisições")  
- `grpms_backend_request()`: Requisição POST com body JSON (VERIFICADO :58)  
- `grpms_current_user_backend_token()`: Obtém token JWT para usuário atual (VERIFICADO :74)  
- `grpms_refresh_dashboard_token()`: AJAX handler para renovar token (VERIFICADO linha 88-110)  
**REST API (grpms-rest-api.php):** (VERIFICADO: namespace `grpms/v1` :6-8; rotas registradas a partir de :908: `/me`, `/profile`, `/profile/password`, `/dashboard/summary`, `/seller/products`, `/seller/orders`, `/marketplace/products`, `/affiliate/commissions`, `/affiliate/affiliations`, `/financial/summary`, `/financial/payment-history`)  
- `grpms_rest_namespace()`: Retorna namespace `grpms/v1` (grpms-rest-api.php:6)  
- `grpms_rest_success()`: Resposta de sucesso padronizada (:10)  
- `grpms_rest_error()`: Resposta de erro padronizada (:18)  
- `grpms_rest_is_error_response()`: Verificação de erro (:30)  
- `grpms_rest_current_user_or_error()`: Obtém usuário atual ou erro (:34)  
- `grpms_rest_user_has_role()`: Verifica se usuário tem role (:47)  
- `grpms_rest_can_access()`: Verifica acesso a funcionalidade (:51)  
- `grpms_rest_pagination()`: Paginação para endpoints (:76)  
- `grpms_rest_table_exists()`: Verifica existência de tabela (:91)  
- `grpms_rest_table_has_column()`: Verifica existência de coluna (:97)  
  
---  
**1.3. Módulos de Acesso e Controle**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - Access Control: VERIFICADO shortcode `grpms_access_control` e tela de gestão (grpms-access-control.php:4, :89, :159).
> - Access Capability: VERIFICADO capability `grpms_manage_access` concedida a `shop_manager` e removida de `administrator` (class-grpms-access-capability.php:4-18); hook `init` VERIFICADO (frontend/grpms-main.php:1565); hook `plugins_loaded`: NAO VERIFICADO (não localizado).
> - Access Validator: VERIFICADO `GRPMS_Access_Validator::check($function)` (class-grpms-access-validator.php:12), usado em grpms-rest-api.php:58 e grpms-backend-client.php:99-100.
> - Access Guard: VERIFICADO `authorize()` (class-grpms-access-guard.php:5); "proteção de rotas": a classe tem 15 linhas, uso por rota NAO VERIFICADO.
> - Access Audit: VERIFICADO `warning`, `denied`, `log` (class-grpms-access-audit.php:5, :17, :31).
> - Access Service: VERIFICADO leitura de permissões por função na tabela de acesso (`get_function_permissions`, class-grpms-access-service.php:16-20).
> - Access Floor: VERIFICADO "piso" de roles mínimas por chave de função (`FLOORS`/`PAYEES`, class-grpms-access-floor.php:5-27); a descrição "Controle de piso de acesso" é confirmada nesse sentido.
> - Access Normalizer: VERIFICADO normalização de linhas/roles e checagem de collation (class-grpms-access-normalizer.php:8-54).
> - Create/Save/Delete AJAX: VERIFICADO ações `wp_ajax_grpms_create_access`, `_save_access`, `_delete_access` (class-grpms-access-create-ajax.php:6, class-grpms-access-save-ajax.php:6, class-grpms-access-delete-ajax.php:8).
> - TOTP Login Gate: VERIFICADO classe `GRPMS_TOTP_Login_Gate` com ação `wp_ajax_grpms_totp_login_verify` (class-grpms-totp-login-gate.php:6, :19); serviço `GRPMS_TOTP_Service` (class-grpms-totp-service.php:6) não estava catalogado.
> - User Session Audit: VERIFICADO captura de login, falha de login e logout (class-grpms-user-session-audit.php:31, :91, :145).
> - Gen Spinner: VERIFICADO spinner de carregamento (class-grpms-gen-spinner.php:6-33).
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Access Control** |  Controle de acesso principal |  - |  `includes/admin/access/grpms-access-control.php` |  - |  
| **Access Capability** |  Definição de capabilities customizadas |  `GRPMS_Access_Capability::ensure()` |  `includes/admin/access/classes/class-grpms-access-capability.php` |  Hook: init, plugins_loaded |  
| **Access Validator** |  Validação de acesso |  `GRPMS_Access_Validator::check()` |  `includes/admin/access/classes/class-grpms-access-validator.php` |  - |  
| **Access Guard** |  Proteção de rotas |  - |  `includes/admin/access/classes/class-grpms-access-guard.php` |  - |  
| **Access Audit** |  Auditoria de acesso |  - |  `includes/admin/access/classes/class-grpms-access-audit.php` |  - |  
| **Access Service** |  Serviço de gestão de acessos |  - |  `includes/admin/access/classes/class-grpms-access-service.php` |  - |  
| **Access Floor** |  Controle de piso de acesso |  - |  `includes/admin/access/classes/class-grpms-access-floor.php` |  - |  
| **Access Normalizer** |  Normalização de dados |  - |  `includes/admin/access/classes/class-grpms-access-normalizer.php` |  - |  
| **Access Create AJAX** |  Handler AJAX para criação de acesso |  - |  `includes/admin/access/classes/class-grpms-access-create-ajax.php` |  Action: wp_ajax_grpms_create_access |  
| **Access Save AJAX** |  Handler AJAX para salvamento de acesso |  - |  `includes/admin/access/classes/class-grpms-access-save-ajax.php` |  Action: wp_ajax_grpms_save_access |  
| **Access Delete AJAX** |  Handler AJAX para exclusão de acesso |  - |  `includes/admin/access/classes/class-grpms-access-delete-ajax.php` |  Action: wp_ajax_grpms_delete_access |  
| **TOTP Login Gate** |  Gate de login com TOTP |  - |  `includes/general/classes/class-grpms-totp-login-gate.php` |  - |  
| **User Session Audit** |  Auditoria de sessões |  - |  `includes/general/classes/class-grpms-user-session-audit.php` |  - |  
| **Gen Spinner** |  Spinner global |  - |  `includes/general/classes/class-grpms-gen-spinner.php` |  - |  
  
   
**Access Control (grpms-access-control.php):**  
- Registra capabilities customizadas via `GRPMS_Access_Capability`  
- Inicialização: `add_action('init', ['GRPMS_Access_Capability', 'ensure'], 1)` (frontend/grpms-main.php:1565; corrigido)  
  
---  
**1.4. Módulos de Afiliação**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - Affiliation: VERIFICADO criação de links por `generate_affiliate_hash` (grpms-affiliation.php:75), botão de afiliação em produto (:10-11), AJAX `add_affiliation`/`disaffiliate_product` (:89, :754), relatório `affiliate_products_report` (:306, :545; shortcode registrado duas vezes).
> - Affiliate General: VERIFICADO captura de visitas (`capture_affiliate_visit`, grpms-affiliategeneral.php:35, :87, hooks :34 e :123 — função declarada duas vezes), vínculo pedido↔visita em `woocommerce_thankyou` (:127-128), shortcodes `affiliate_visits` (:508) e `recent_commissions` (:536).
> - Affiliate Dashboard: VERIFICADO shortcode `affiliate_dashboard` (grpms-affiliate-dashboard.php:8) que consome `/api/affiliate-dashboard` (:120) e AJAX financeiro (:145).
> - Commissions (`includes/grpms-commissions.php`): **a descrição "Cálculo de comissões" é INCORRETA**. O arquivo tem 11 linhas e apenas imprime `the_content()` (VERIFICADO grpms-commissions.php:1-11). A percentagem da plataforma é gravada em `frontend/includes/admin/platform/grpms-platform.php` (opção `grpms_platform_commission`, :42, :129) e a comissão do afiliado por produto vem de metadados (`af_commission`, grpms-affiliation.php:197); o cálculo aplicado ao pedido está no backend (backend/fastapi_app/services/checkout/order_service.py:156 `save_commissions`; backend/fastapi_app/services/checkout/commission_math.py:4, somente com `GRPMS_COMMISSION_WATERFALL`).
> - "Validação de hashes de afiliados" no módulo PHP: NAO VERIFICADO no plugin; a validação está no backend (checkout_service.py:41) e é chamada por grpms-checkout.php:1287 e grpms-guest-shipping.php:218.
> - "Integração com YITH Affiliates": o código referencia apenas a role/capability `yith_affiliate` (VERIFICADO grpms-affiliation.php:17, grpms-affiliate-dashboard.php:18, grpms-affiliategeneral.php:169; backend checkout_service.py:24). Nenhuma chamada a classes/funções do plugin YITH foi localizada; o plugin em si: NAO VERIFICADO.
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Affiliation** |  Gerenciamento de afiliados |  - |  `includes/affiliation/grpms-affiliation.php` |  - |  
| **Affiliate General** |  Funções gerais de afiliação |  - |  `includes/affiliation/grpms-affiliategeneral.php` |  - |  
| **Affiliate Dashboard** |  Dashboard do afiliado |  - |  `includes/affiliation/grpms-affiliate-dashboard.php` |  - |  
| **Commissions** |  Cálculo de comissões |  - |  `includes/grpms-commissions.php` |  - |  
  
   
**Affiliation (grpms-affiliation.php):**  
- Gerenciamento de links de afiliados  
- Validação de hashes de afiliados  
- Integração com YITH Affiliates (NAO VERIFICADO)  
**Commissions (grpms-commissions.php):**  
- Cálculo de comissões para vendedores e afiliados  
- Distribuição de valores entre partes  
  
---  
**1.5. Módulos de Checkout**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - Checkout (`grpms-checkout.php`, 2354 linhas): VERIFICADO shortcodes `grpms_shipping_calculator` (:848), `grpms_vendor_checkout` (:928), `grpms_vendor_message` (:953), `grpms_vendor_logo` (:984); validação de CEP via AJAX (:865-866); validação de afiliado (:1281-1287); conversão de moeda (:1439-1440); envio do pedido ao backend `/api/checkout/save-order` (:2029); validação no checkout (:2108).
> - "Integração com gateways de pagamento": NAO VERIFICADO como integração; o código só detecta ids de gateway `paypal`/`ppcp` e filtra moedas (VERIFICADO :1097-1125). Nenhuma chamada a API de gateway.
> - "Processamento de pedidos": o plugin envia o pedido ao backend (VERIFICADO :2029); o processamento ocorre no backend (backend/fastapi_app/routers/checkout/order_router.py:11, backend/fastapi_app/services/checkout/order_service.py:504).
> - Checkout Guard: VERIFICADO bloqueio da conclusão por etiqueta de frete (`grpms_has_label_purchased`, `grpms_label_pending`, filtro `woocommerce_payment_complete_order_status`, hook `template_redirect`; grpms-checkout-guard.php:5-73), consultando `grpms_easypost_shipments`.
> - Checkout Thank You: VERIFICADO shortcode `grpms_vendor_contact_info` (grpms-checkout-thankyou.php:7-8).
> - Shipping Calculator (arquivo `includes/checkout/grpms-shipping-calculator.php`): VERIFICADO page template "Shipping Calculator" de 10 linhas que executa `[grpms_shipping_calculator]` (:1-10).
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Checkout** |  Lógica customizada de checkout |  - |  `includes/checkout/grpms-checkout.php` |  - |  
| **Checkout Guard** |  Proteção de checkout |  - |  `includes/checkout/grpms-checkout-guard.php` |  - |  
| **Checkout Thank You** |  Página de agradecimento |  - |  `includes/checkout/grpms-checkout-thankyou.php` |  - |  
| **Shipping Calculator** |  Calculadora de frete |  - |  `includes/checkout/grpms-shipping-calculator.php` |  Template: grpms-shipping-calculator.php |  
  
   
**Checkout (grpms-checkout.php):**  
- Validação de dados de checkout  
- Integração com gateways de pagamento (NAO VERIFICADO)  
- Processamento de pedidos  
**Shipping Calculator:**  
- Template customizado registrado via `theme_page_templates` filter (VERIFICADO frontend/grpms-main.php:1518)  
- Carregado via `template_include` filter (VERIFICADO frontend/grpms-main.php:1524)  
  
---  
**1.6. Módulos de Dashboard**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - Dashboard (`includes/dashboard/grpms-dashboard.php`): a tag "Shortcode: [grp_dashboard]" é **NAO VERIFICADA**: o arquivo registra `grp_sales_chart`, `grp_top_products_chart` e `grp_affiliate_earnings_chart` (VERIFICADO :13, :26, :39) e ações `wp_ajax_grp_dashboard_data`/`grp_dashboard_statistics` (:75-76) que fazem proxy ao backend com JWT (`grp_dashboard_api_get`, :152); o shortcode `grp_dashboard` é apenas consultado por `shortcode_exists` em frontend/grpms-main.php:1695 e **não é registrado** em `includes/`.
> - "Exibição de estatísticas para vendedores e afiliados": VERIFICADO contexto vendor/affiliate (grpms-dashboard.php:86-107) e estatísticas locais (:218).
> - Dash (`grpms-dash.php`): VERIFICADO shortcode `grpms_dash` (:8), logout AJAX (:535-537) e busca global de produtos/pedidos/usuários (:561-731); "Funções do dash" é descrição genérica, NAO VERIFICADA como contrato.
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Dashboard** |  Dashboard principal |  - |  `includes/dashboard/grpms-dashboard.php` |  Shortcode: \[grp_dashboard\] |  
| **Dash** |  Funções do dash |  - |  `includes/dash/grpms-dash.php` |  - |  
  
   
**Dashboard (grpms-dashboard.php):**  
- Exibição de estatísticas para vendedores e afiliados  
- Integração com backend para obtenção de dados  
- Shortcode: `[grp_dashboard]` (consultado em frontend/grpms-main.php:1695; NAO VERIFICADO: não registrado em `includes/`; ver nota acima)  
  
---  
**1.7. Módulos Financeiros**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - Commissions: ver 1.4 — o arquivo `grpms-commissions.php` NÃO calcula comissões (11 linhas, VERIFICADO). Observação: o caminho listado abaixo, `grpms-commissions.php`, é relativo a `includes/`.
> - Payment (`grpms-payment.php`, 2869 linhas): a descrição "Processamento de pagamentos" é **INCORRETA/imprecisa**: o módulo gerencia solicitações de saque/repasse de vendedores e afiliados (VERIFICADO shortcodes `grpms_payment` :148, `grpms_requests_list` :1018, `grpms_admin_requests_list` :1038; AJAX `process_payment_request` :724, `check_paypal_set` :689, `filter_status` :530; valor mínimo por moeda :37-53). Não foi localizada cobrança de comprador nem chamada a API de pagamento.
> - PayPal Set: VERIFICADO formulário que grava o e-mail PayPal do vendedor/afiliado (`save_paypal_email` :20, shortcode `paypal_email_form` :245). "Configuração PayPal" refere-se a esse e-mail de repasse; nenhuma chamada à API PayPal. O gateway PayPal do checkout: NAO VERIFICADO (plugin externo).
> - Admin Request Review (`grpms-admin-request-review.php`, não catalogado): VERIFICADO função `grpms_commit_admin_request_review` (:5).
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Commissions** |  Cálculo de comissões |  - |  `grpms-commissions.php` |  - |  
| **Payment** |  Processamento de pagamentos |  - |  `includes/financial/grpms-payment.php` |  - |  
| **PayPal Set** |  Configuração PayPal |  - |  `includes/financial/grpms-paypalset.php` |  - |  
  
   
  
---  
**1.8. Módulos de Galeria**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - Gallery: VERIFICADO shortcode `product_gallery` (grpms-gallery.php:45), AJAX com e sem login (`grpms_get_image`, `grpms_gallery_products`, `grpms_gallery_search`; :48-49, :121-124) que fazem proxy ao backend (`grpms_gallery_api_get`, :177), com nonce (:126) e rate limit (:140).
> - Gallery Catalog / Gallery Image: VERIFICADO enriquecimento de catálogo (`grpms_gallery_enrich_catalog`, grpms-gallery-catalog.php:5) e URLs de imagem (`grpms_gallery_image_url`, grpms-gallery-image.php:5), não catalogados antes.
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Gallery** |  Gerenciamento de galeria de produtos |  - |  `includes/gallery/grpms-gallery.php` |  - |  
  
   
  
---  
**1.9. Módulos Gerais**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - General (`grpms-general.php`, 1734 linhas): "Funções utilitárias gerais" é descrição genérica. VERIFICADO conteúdo: shortcode `tuzzello_buttons` com assistente de ajuda/modal (:55-56, :515) e `user_real_name` (:753); estilos de modal de produto (:9-18); 41 hooks.
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **General** |  Funções utilitárias gerais |  - |  `includes/general/grpms-general.php` |  - |  
  
   
**Classes em includes/general/classes/:**  
- `GRPMS_Gen_Spinner` (VERIFICADO class-grpms-gen-spinner.php:6)
- Outras classes VERIFICADAS (não catalogadas antes): `GRPMS_Gen_AdminBar` (oculta a barra de admin para não administradores, class-grpms-gen-adminbar.php:6-15), `GRPMS_Gen_Assets`, `GRPMS_Gen_Buttons`, `GRPMS_Gen_Login_Redirect` (:7-55), `GRPMS_Gen_Mail` (configura SMTP, :17), `GRPMS_Gen_Shipping` (agenda atualização diária de status de frete, `wp_schedule_event`, class-grpms-gen-shipping.php:64), `GRPMS_Gen_Uploads`, `GRPMS_Gen_User_Profile`, `GRPMS_TOTP_Service`; `class-grpms-gen-orders.php` tem 3 linhas (conteúdo não relevante)  
  
---  
**1.10. Módulos de Perfil**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - Profile (`grpms-profile.php`): VERIFICADO shortcode `custom_profile` (:70), troca de senha (:574), atualização de perfil com upload de documentos KYC (:623-667) e fluxo TOTP (:966, :989, :1033).
> - Profile KYC Expiry: VERIFICADO `grpms_kyc_valid_expiry`/`grpms_kyc_expiry_blocked` (profile-kyc-expiry.php:4, :10).
> - Não catalogados antes e VERIFICADOS: `profile-kyc-files.php` (armazenamento privado e entrega de arquivos KYC, :26-182), `profile-postal.php` (consulta de CEP, :30, :100).
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Profile** |  Gerenciamento de perfil |  - |  `includes/profile/grpms-profile.php` |  - |  
| **Profile KYC Expiry** |  Expiração de KYC no perfil |  - |  `includes/profile/profile-kyc-expiry.php` |  - |  
  
   
**Classes em includes/profile/classes/:**  
- VERIFICADO: existe `class-grpms-profile-assets.php` (carrega CSS/JS do perfil, :6-43); outras classes: nenhuma (autoload varre o diretório, frontend/grpms-main.php:1537)  
  
---  
**1.11. Módulos de Produtos**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - Products (`grpms-products.php`, 7181 linhas): VERIFICADO listagem do vendedor (`load_vendor_products` :270), formulário e salvamento de edição (`edit_product_custom` :604, `save_product_changes` :2184), alternância de status (:2295/:2303, handler registrado duas vezes) e snapshot de aprovação de alterações (:175-230).
> - Product Display: VERIFICADO shortcode `product_display` (grpms-productdisplay.php:512) e AJAX `add_affiliation` (:604, também registrado em grpms-affiliation.php:89).
> - Products Approval: VERIFICADO fila de aprovação com ações de aprovar/rejeitar e revisão de treinamento (grpms-products-approval.php:577-619).
> - Product Delete: VERIFICADO shortcode `manage_product_deletions` e cancelamento de exclusão (grpms-product-delete.php:11, :73); aprovação de exclusão em `grpms_product_deletion_can_approve` (:13).
> - Não catalogados antes e VERIFICADOS: `grpms-public-preview.php` (oculta produtos de consultas públicas e REST, :65, :76), `product-text-limits.php` (:5).
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Products** |  Gerenciamento de produtos |  - |  `includes/products/grpms-products.php` |  - |  
| **Product Display** |  Exibição de produtos |  - |  `includes/products/grpms-productdisplay.php` |  - |  
| **Products Approval** |  Aprovação de produtos |  - |  `includes/admin/products_approval/grpms-products-approval.php` |  - |  
| **Product Delete** |  Exclusão de produtos |  - |  `includes/admin/delete_products/grpms-product-delete.php` |  - |  
  
   
  
---  
**1.12. Módulos de Vendedor**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - Seller (`grpms-seller.php`, 1557 linhas): VERIFICADO relatório de afiliados do vendedor (:145, :692 duplicado), cancelamento de afiliação (:436, :606), status de frete (:1092-1094), travas de pedido (`grpms_install_order_locks` :1181) e e-mails a vendedor/afiliado (:1210); "Gerenciamento de vendedores" é descrição ampla, NAO VERIFICADA como contrato.
> - Seller Dashboard: o componente não tem `Path` na tabela; VERIFICADO em `includes/seller/grpms-seller-dashboard.php` (shortcode `seller_dashboard` :9, consome `/api/seller-dashboard` :171). Router backend VERIFICADO em main.py:37.
> - Não catalogados antes e VERIFICADOS: `grpms-seller-orders.php` (`grpms_seller_orders` :121, AJAX `load_orders` :124), `grpms-seller-store-config.php` (configuração de checkout do vendedor, :417-425), `grpms-shipping-config.php` (zonas de frete do vendedor, AJAX :210-423; tabela `grpms_user_shipping_zone` é lida por shipping_service.py:5-30).
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Seller** |  Gerenciamento de vendedores |  - |  `includes/seller/grpms-seller.php` |  - |  
| **Seller Dashboard** |  Dashboard do vendedor |  - |  - |  Router: /api/seller-dashboard |  
  
   
  
---  
**1.13. Módulos de Treinamento**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - Training: VERIFICADO shortcodes `grpms_vendor_training` e `grpms_training_settings` (grpms-training.php:314-315) e ~30 ações AJAX (:331-); classes `GRPMS_Training_Seller`, `_Student`, `_Settings`, `_Enrollment`, `_CPT`, `_Product`, `_Bunny` em `includes/training/classes/`. Matrícula reage a pagamento concluído/reembolso (`handle_payment_complete`, `handle_order_refunded`, class-grpms-training-enrollment.php:15-23). Certificado em PDF (class-grpms-training-student.php:19, :182). Vídeos via classe Bunny (class-grpms-training-bunny.php:44-123); o serviço Bunny em si: NAO VERIFICADO (externo).
> - `grpms-training-review.php`: VERIFICADO fila de revisão de currículo (:5-91).
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Training** |  Área de treinamento |  - |  `includes/training/grpms-training.php` |  - |  
  
   
**Role de treinamento:**  
- `tuzzello_student` registrada em frontend/grpms-main.php:13-21  
  
---  
**1.14. Módulos de PWA**  
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **PWA** |  Progressive Web App |  - |  `includes/pwa/grpms-pwa.php` |  - |  
  
   
**Funcionalidades:**  
- Detecção de path `/app` (VERIFICADO frontend/grpms-main.php:48-52)  
- Login PWA (VERIFICADO frontend/grpms-main.php:80-127 e :1357-1362)  
- Módulo PWA (`grpms-pwa.php`, 1157 linhas): VERIFICADO cookie de intenção de login (:10-28), `grpms_pwa_render_manifest` (:315), service worker (`grpms_pwa_render_service_work...`, :358), logout (:294); descrição "Progressive Web App" confirmada nesse sentido  
  
---  
**1.15. Módulos de Plataforma**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - Platform (`frontend/includes/admin/platform/grpms-platform.php`): VERIFICADO shortcode `grpms_platform_commission` que grava a percentagem de comissão da plataforma na opção `grpms_platform_commission` (:8, :42, :129) via AJAX `save_grpms_commission` (:114); "Configurações da plataforma" confirmado apenas para a comissão (e validação de sessão, :147).
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Platform** |  Configurações da plataforma |  - |  `frontend/includes/admin/platform/grpms-platform.php` |  - |  
  
   
**Classes em includes/admin/platform/classes/:**  
- VERIFICADO: o diretório `includes/admin/platform/classes/` não existe neste HEAD (apenas `frontend/includes/admin/platform/grpms-platform.php`); o autoload referencia o caminho (frontend/grpms-main.php:1537)  
  
---  
**1.16. Módulos de KYC**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - KYC (`grpms-kyc.php`): VERIFICADO tabela própria (`grpms_kyc_ensure_table` :10), shortcode de revisão `grpms_kyc_review` (:285), atualização de status via AJAX (:387) e carga de registros (:460). O backend consome `kyc_last_status = 'approved'` do afiliado (checkout_service.py:6-24).
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **KYC** |  Verificação de identidade |  - |  `includes/admin/kyc/grpms-kyc.php` |  - |  
  
   
  
---  
**1.17. Templates**  

> **Verificação por afirmação (ARCH-FIX-03):**
> - "Add Product Template" com Path `includes/checkout/grpms-shipping-calculator.php`: **INCORRETO**. O template Add Product é `templates/template-add-product.php`, selecionado quando `?grp_embed=1` em `/mydash/t-v-addprod` (VERIFICADO frontend/grpms-main.php:1464-1467); o conteúdo desse arquivo de template: NAO VERIFICADO. `includes/checkout/grpms-shipping-calculator.php` é o page template "Shipping Calculator" (VERIFICADO, 10 linhas).
> - South Africa Landing: VERIFICADO roteamento para `/html/south-africa/index.php` (frontend/grpms-main.php:688, :703).
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Add Product Template** |  Template para adição de produtos |  - |  `templates/template-add-product.php` (corrigido; VERIFICADO frontend/grpms-main.php:1466) |  - |  
| **South Africa Landing** |  Landing page África do Sul |  - |  `html/south-africa/index.php` |  - |  
  
   
**Template Add Product:**  
- Registrado para path `/mydash/t-v-addprod` (VERIFICADO frontend/grpms-main.php:1464)  
- Carregado quando `?grp_embed=1` (VERIFICADO frontend/grpms-main.php:1465)  
  
---  
**1.18. Arquivos Estáticos**  
  
|  |  |  |  |  
| - | - | - | - |  
| **Componente** |  **Tipo** |  **Path** |  **Descrição** |  
| **CSS** |  CSS |  `css/` |  Estilos do plugin |  
| **JS** |  JavaScript |  `js/` |  Scripts do plugin |  
| **Images** |  Imagens |  NAO VERIFICADO |  - |  

   

---  
**1.19. Módulos adicionais carregados e antes não catalogados**  

> Todos VERIFICADOS por leitura do código e do carregamento em frontend/grpms-main.php:1480-1508.

- `includes/registry/grpms-registry.php`: cadastro de usuário (shortcode `grpms_registry` :135; AJAX `grpms_register_user` com e sem login :138-139), lista de ativação (`user_activation_list` :510), alternância de status (:967), ação TOTP de admin (:1014)  
- `includes/admin/tax_liability/grpms-tax-liability.php`: alíquotas por país (shortcode `grpms_tax_liability` :71; AJAX :73, :79)  
- `includes/admin/approve_orders/grpms-approve_orders.php`: aprovação manual de pedidos (shortcode `manage_processing_orders` :9; transições completed/failed com auditoria e rate limit, :117-364)  
- `includes/admin/orders_history/grpms-orders-history.php`: histórico de pedidos com filtros e gráfico (shortcode `grpms_orders_history` :603; AJAX :605, :614)  
  
   
  
---  
**2. Backend FastAPI**  
**2.1. Aplicação Principal**  
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **Main App** |  Aplicação FastAPI principal |  `app` |  `fastapi_app/main.py` |  - |  
  
   
**Main (main.py):**  
- Importação de routers (VERIFICADO main.py:9-21)  
- Criação da app FastAPI (VERIFICADO main.py:23)  
- Middleware CORS (VERIFICADO main.py:26-32; origens vindas de `CORS_ALLOWED_ORIGINS`, padrão com um único domínio)  
- Registro de routers (VERIFICADO main.py:24, 35-48)  
  - woocommerce_order.router  
  - token_router.router  
  - dashboard_main.router (prefix=/api/dashboard-main)  
  - seller_dashboard_router.router (prefix=/api/seller-dashboard)  
  - affiliate_dashboard_router.router (prefix=/api/affiliate-dashboard)  
  - product_gallery.router  
  - product_gallery_search.router  
  - checkout_router.router  
  - shipping_router.router  
  - easypost_router (prefix=/api)  
  - order_router.router  
  - mobile_sales_router  
  - mobile_profile_router  
  
---  
**2.2. Módulos de Autenticação**  
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Tags** |  
| **JWT RS256** |  Implementação JWT com RS256 |  `create_token()`, `verify_token()` |  `fastapi_app/auth/jwt_rs256.py` |  - |  
| **WordPress Bridge** |  Bridge de autenticação com WordPress |  `require_wordpress()` |  `fastapi_app/auth/wordpress_bridge.py` |  Header: X-GRPMS-WP-Secret |  
| **Identity** |  Identidade do usuário |  - |  `fastapi_app/auth/identity.py` |  - |  
| **Mobile Auth** |  Autenticação móvel |  - |  `fastapi_app/auth/mobile.py` |  - |  
| **Dependencies** |  Dependências de autenticação |  - |  `fastapi_app/auth/dependencies.py` |  - |  
  
   
**WordPress Bridge (wordpress_bridge.py):**  
- `require_wordpress()`: Dependency que valida header `X-GRPMS-WP-Secret` (VERIFICADO linha 10-15; comparação em :14)
- `require_vendor_identity()`: identidade de vendedor assinada por HMAC (`X-GRPMS-Vendor-Id/Ts/Sig`, janela de 60 s) (VERIFICADO wordpress_bridge.py:18-35)  
  - Verifica se segredo está configurado (GRPMS_WP_SHARED_SECRET)  
  - Comparação segura via `secrets.compare_digest()`  
  - Usado como dependência em rotas protegidas  
**JWT RS256 (jwt_rs256.py):**  
- `create_token()`: Cria token JWT com payload (VERIFICADO jwt_rs256.py:19-23; algoritmo RS256 :16; expiração `JWT_EXPIRE_MINUTES`, padrão 15, :17)  
- `verify_token()`: Verifica token JWT (VERIFICADO jwt_rs256.py:25-27)  
- Usado em: `token_router.py:5` (import), `token_router.py:35` (chamada)  
**Dependencies (dependencies.py):**  
- `get_current_user()`: Obtém usuário atual do token JWT (VERIFICADO dependencies.py:9-17)  
- `get_current_user_optional()`: Obtém usuário atual (opcional) (VERIFICADO dependencies.py:20-)  
- `identity.py` e `mobile.py` (descrição "Identidade do usuário" e "Autenticação móvel"): NAO VERIFICADO (conteúdo não detalhado); `mobile.py` fornece `require_mobile_user_id`, usado em routers/mobile/mobile_salehistory_router.py:13 (VERIFICADO)  
  
---  
**2.3. Routers**  
***2.3.1. Routers de Checkout***

> **Verificação por afirmação (ARCH-FIX-03):** `main.py:24` registra o webhook WooCommerce antes do CORS (VERIFICADO). Shipping Router: VERIFICADO `GET /get-cost` com `require_wordpress` (shipping_router.py:10, :15), que chama `get_shipping_cost` (shipping_service.py:5-30, lê `grpms_user_shipping_zone`). Order Router: VERIFICADO `POST /api/checkout/save-order` (backend/fastapi_app/routers/checkout/order_router.py:11) e `POST /api/checkout/save-addresses` (:23), ambos com `require_wordpress` (:8). Dashboard Main (`/statistics`, `/dashboard/data`), Seller (`/data`), Affiliate (`/data`): VERIFICADO e autenticados por JWT (`get_current_user`; dashboard_main.py:11, seller_dashboard_router.py:14, affiliate_dashboard_router.py:11). Galeria: VERIFICADO `GET /product-gallery` e `GET /product-search` (product_gallery.py:18, product_gallery_search.py:16). Mobile: VERIFICADO `GET /profile` e `GET /sales-history`.  
  
|  |  |  |  |  |  |  
| - | - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Prefix** |  **Tags** |  
| **Checkout Router** |  Operações de checkout |  `router` |  `fastapi_app/routers/checkout/checkout_router.py` |  /api/checkout |  WordPress bridge |  
| **Shipping Router** |  Cálculo de frete |  `router` |  `fastapi_app/routers/checkout/shipping_router.py` |  - |  - |  
| **Order Router** |  Gestão de pedidos |  `router` |  `backend/fastapi_app/routers/checkout/order_router.py` |  - |  - |  
| **EasyPost Router** |  Integração EasyPost |  `router` |  `fastapi_app/routers/checkout/easypost_router.py` |  /api |  EasyPost |  
  
   
**Checkout Router (checkout/checkout_router.py):**  
- `validate_affiliate()`: Valida link de afiliado (VERIFICADO checkout_router.py:27-43)  
  - GET /api/checkout/validate-affiliate  
  - Parâmetros: hash, product_id  
  - Valida via `checkout_service.validate_affiliate_link()`  
  - Dependências: `require_wordpress`, `get_current_user_optional`  
**EasyPost Router (checkout/easypost_router.py):**  
- Rotas EasyPost (VERIFICADO easypost_router.py): `create_address` :53, `create_shipment` :68, `buy_shipment` :83 (503 sem a compra habilitada, :85), `track` :88, `tracker-status` :104, `shipment-history` :126, `calculate-shipping` :141, `validate-selection` :161, `webhooks/easypost` :190 (503, :192)  
- Dependências: `require_wordpress` (VERIFICADO :16) e `require_vendor_identity` nas rotas de rastreio (:3, :89, :105, :127)  
***2.3.2. Routers de Dashboard***  
  
|  |  |  |  |  |  |  
| - | - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Prefix** |  **Tags** |  
| **Dashboard Main** |  Dashboard principal |  `router` |  `fastapi_app/routers/dashboard_main.py` |  /api/dashboard-main |  DashboardMain |  
| **Seller Dashboard** |  Dashboard do vendedor |  `router` |  `fastapi_app/routers/seller_dashboard_router.py` |  /api/seller-dashboard |  SellerDashboard |  
| **Affiliate Dashboard** |  Dashboard do afiliado |  `router` |  `fastapi_app/routers/affiliate_dashboard_router.py` |  /api/affiliate-dashboard |  AffiliateDashboard |  
  
   
***2.3.3. Routers de Galeria de Produtos***  
  
|  |  |  |  |  |  |  
| - | - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Prefix** |  **Tags** |  
| **Product Gallery** |  Galeria de produtos |  `router` |  `fastapi_app/routers/product_gallery.py` |  - |  - |  
| **Product Gallery Search** |  Busca na galeria |  `router` |  `fastapi_app/routers/product_gallery_search.py` |  - |  - |  
  
   
***2.3.4. Routers Móveis***  
  
|  |  |  |  |  |  |  
| - | - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Prefix** |  **Tags** |  
| **Mobile Profile** |  Perfil móvel |  `router` |  `fastapi_app/routers/mobile/mobile_profile_router.py` |  - |  - |  
| **Mobile Sales History** |  Histórico de vendas móvel |  `router` |  `fastapi_app/routers/mobile/mobile_salehistory_router.py` |  - |  - |  
  
   
***2.3.5. Router de Token***  
  
|  |  |  |  |  |  |  
| - | - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Prefix** |  **Tags** |  
| **Token Router** |  Geração de tokens JWT |  `router` |  `fastapi_app/routers/token_router.py` |  - |  - |  
  
   
**Token Router (token_router.py):**  
- `retired_token_endpoint()`: Endpoint deprecated (VERIFICADO token_router.py:13-16)  
  - POST /api/token e POST /api/token/login — **aposentados, retornam HTTP 410** (token_router.py:13-16)  
  - Retorna 410 Gone  
- `generate_token_from_wp()`: Gera token JWT para usuário WordPress (VERIFICADO token_router.py:18-42)  
  - POST /api/token/from-wp  
  - Dependências: `require_wordpress`  
  - Input: `WPTokenRequest` (user_id: int)  
  - Acesso ao BD: Consulta `wp_usermeta` para `wp_capabilities`  
  - Parse: `phpserialize.loads()` para deserialize capabilities  
  - Output: `{"access_token": create_token(...)}`  
***2.3.6. Routers de Webhooks***  
  
|  |  |  |  |  |  |  
| - | - | - | - | - | - |  
| **Componente** |  **Descrição** |  **Entry Point** |  **Path** |  **Prefix** |  **Tags** |  
| **WooCommerce Order** |  Webhook assinado de pedidos WooCommerce (`POST /webhooks/woocommerce/order`); **padrão desligado** (`GRPMS_AUTO_COMPLETE`), HMAC com `GRPMS_WC_WEBHOOK_SECRET` |  `router` |  `fastapi_app/routers/webhooks/woocommerce_order.py` |  - |  - |  
  
   
  
---  
**2.4. Services**  
***2.4.1. Services de Checkout***  
  
|  |  |  |  |  
| - | - | - | - |  
| **Componente** |  **Descrição** |  **Path** |  **Dependências** |  
| **Checkout Service** |  Lógica de checkout |  `fastapi_app/services/checkout/checkout_service.py` |  database, models |  
| **EasyPost Service** |  Integração EasyPost |  `fastapi_app/services/checkout/easypost_service.py` |  - |  
| **Order Service** |  Processamento de pedidos |  `backend/fastapi_app/services/checkout/order_service.py` |  database, financial_fee_service, easypost_service |  
| **Shipping Service** |  Lógica de frete |  `fastapi_app/services/checkout/shipping_service.py` |  - |  
| **Financial Fee Service** |  Taxas financeiras |  `fastapi_app/services/checkout/financial_fee_service.py` |  - |  
| **Commission Math** |  Cálculo de comissões |  `backend/fastapi_app/services/checkout/commission_math.py` |  - |  
| **Refund Guard** |  Proteção de reembolsos |  `fastapi_app/services/checkout/refund_guard.py` |  - |  
| **Refund Store** |  Processamento de reembolsos |  `fastapi_app/services/checkout/refund_store.py` |  - |  
| **Auto Complete** |  Auto-conclusão de pedidos |  `fastapi_app/services/checkout/auto_complete.py` |  - |  
| **WC Webhook** |  Verificação/processamento do webhook WooCommerce (padrão desligado); relacionados: `wc_outbox_worker.py` (outbox, sem scheduler instalado) e `auto_complete.py` (flag `GRPMS_AUTO_COMPLETE`, não ligado) |  `fastapi_app/services/checkout/wc_webhook.py` |  - |  
| **WC Client** |  Cliente WooCommerce |  `backend/fastapi_app/services/checkout/wc_client.py` |  - |  
| **WC Outbox Worker** |  Trabalhador de saída |  `fastapi_app/services/checkout/wc_outbox_worker.py` |  - |  
| **Alert Mail** |  Alertas por email |  `fastapi_app/services/checkout/alert_mail.py` |  - |  
  
   
**Order Service (backend/fastapi_app/services/checkout/order_service.py):**  
- `reconciliation()`: Log de reconciliação (VERIFICADO backend/fastapi_app/services/checkout/order_service.py:23)  
- `OrderReconciliationError`: Exceção customizada (VERIFICADO :28)  
- `_num()`: Validação de valores numéricos (VERIFICADO :32)  
- `canonical_items()`: Validação de itens do pedido (VERIFICADO :42-84)  
  - Compara itens da requisição com `wp_woocommerce_order_items`/`itemmeta` (VERIFICADO :44-60)  
  - Verifica product_id, quantity e `_line_total` (VERIFICADO :60-76)  
  - Em divergência registra e lança `RuntimeError` (VERIFICADO :80-84); a classe `OrderReconciliationError` é tratada por backend/fastapi_app/routers/checkout/order_router.py:15 (conversão entre as duas: NAO VERIFICADO)  
- Também VERIFICADO: `platform_percentage` (:86), `verified_order_link` (:99), `commission_waterfall_enabled` (:133, padrão desligado), `save_commissions` (:156), `insert_order_into_main_tables` (:504), `insert_order_address` (:419), `log_order_status_change` (:466)  

**Demais services de checkout (ARCH-FIX-03, todos lidos):**  
- Checkout Service (VERIFICADO checkout_service.py:6, :28, :41): elegibilidade de afiliado (role `yith_affiliate` + `kyc_last_status = approved`), elegibilidade de produto e `validate_affiliate_link`. Dependências reais: `database`, `phpserialize` (o catálogo listava "database, models"; `models`: NAO VERIFICADO)  
- EasyPost Service (VERIFICADO easypost_service.py:28-518): classe `EasyPostService` sobre o SDK `easypost.EasyPostClient` (:10) com `calculate_shipping` :353, `validate_selection` :464, `buy_shipment_with_rate` :128, `track` :217, `save_shipment_to_db` :267. Dependências reais: `database` (:11-12); a coluna "Dependências: -" estava incompleta  
- Shipping Service (VERIFICADO shipping_service.py:5-30): custo de frete do vendedor por país a partir de `grpms_user_shipping_zone`, com fallback "OT"; não usa EasyPost  
- Financial Fee Service (VERIFICADO financial_fee_service.py:3): `save_woopayment_fee_hpos` lê metadados `_wcpay_payment_transaction_id` e `_wcpay_transaction_fee` em `wp_wc_orders_meta`; não chama Stripe nem plugin de pagamento, apenas lê metadados que o plugin de pagamento do WooCommerce teria gravado (Stripe via plugin do WooCommerce segundo o PO; identidade do plugin, p.ex. WooPayments, e existência: NAO VERIFICADO)  
- Commission Math (VERIFICADO backend/fastapi_app/services/checkout/commission_math.py:4-11): função pura em centavos e pontos-base; usada apenas quando o waterfall está ativo (backend/fastapi_app/services/checkout/order_service.py:199)  
- Refund Guard (VERIFICADO refund_guard.py:10-49): elegibilidade de reembolso sem I/O. Refund Store (VERIFICADO refund_store.py, docstring "Unwired T2A SQL helpers"): não ligado a nenhuma rota. Auto Complete (VERIFICADO auto_complete.py:1-9, "Unwired, default-off")  
- WC Webhook (VERIFICADO wc_webhook.py:21, :52, :126), WC Client (VERIFICADO backend/fastapi_app/services/checkout/wc_client.py:123-, usa `httpx` e `/wp-json/wc/v3/orders/{id}` :140, descrito como "LAB-only"), WC Outbox Worker (VERIFICADO wc_outbox_worker.py:1-6, :128-130: "No scheduler is installed")  
- Alert Mail (VERIFICADO alert_mail.py:1-14): alertas por SMTP somente do worker, desabilitado salvo configuração; usa `smtplib`  
***2.4.2. Services de Dashboard***  
> **Verificação (ARCH-FIX-03):** VERIFICADO `get_vendor_statistics`/`get_affiliate_statistics`/`get_dashboard_statistics`/`get_dashboard_graph_data` (dashboard_main_statistics.py:12, :76, :140, :153), `get_affiliate_dashboard_data` (affiliate_dashboard_service.py:5) e `get_seller_dashboard_data` (seller_dashboard_service.py:6). Os services consultam diretamente o banco; referências a `wcfm`/`yith` em dashboard_main_statistics.py:141-143 são nomes de role/consulta, não chamadas a plugins.  
  
|  |  |  |  |  
| - | - | - | - |  
| **Componente** |  **Descrição** |  **Path** |  **Dependências** |  
| **Dashboard Main Statistics** |  Estatísticas do dashboard principal |  `fastapi_app/services/dashboard_main_statistics.py` |  - |  
| **Affiliate Dashboard Service** |  Serviço de dashboard do afiliado |  `fastapi_app/services/affiliate_dashboard_service.py` |  - |  
| **Seller Dashboard Service** |  Serviço de dashboard do vendedor |  `fastapi_app/services/seller_dashboard_service.py` |  - |  
  
   
***2.4.3. Services de Galeria de Produtos***  
> **Verificação (ARCH-FIX-03):** VERIFICADO `fetch_product_gallery` (product_gallery_service.py:14) e `search_products_by_title` (product_gallery_search_service.py:5); a menção a "WooCommerce" em product_gallery_service.py:39 refere-se a metadados do WooCommerce lidos do banco.  
  
|  |  |  |  |  
| - | - | - | - |  
| **Componente** |  **Descrição** |  **Path** |  **Dependências** |  
| **Product Gallery Service** |  Serviço de galeria de produtos |  `fastapi_app/services/product_gallery_service.py` |  - |  
| **Product Gallery Search Service** |  Serviço de busca na galeria |  `fastapi_app/services/product_gallery_search_service.py` |  - |  
  
   
***2.4.4. Services Móveis***  
> **Verificação (ARCH-FIX-03):** VERIFICADO `get_profile_data` (mobile_profile_service.py:32) e `get_sales_history_data`/`get_sales_summary` (mobile_saleshistory_service.py:15, :153).  
  
|  |  |  |  |  
| - | - | - | - |  
| **Componente** |  **Descrição** |  **Path** |  **Dependências** |  
| **Mobile Profile Service** |  Serviço de perfil móvel |  `fastapi_app/services/mobile/mobile_profile_service.py` |  - |  
| **Mobile Sales History Service** |  Serviço de histórico de vendas móvel |  `fastapi_app/services/mobile/mobile_saleshistory_service.py` |  - |  
  
   
  
---  
**2.5. Models**  
  
|  |  |  |  |  
| - | - | - | - |  
| **Componente** |  **Descrição** |  **Path** |  **Campos Principais** |  
| **ProductOut** |  Schema de produto (saída) |  `fastapi_app/models.py` |  id, title, image, link, price_min, price_max, commission_range, discount_range, vendor_name, affiliate_commission_total, is_digital, is_virtual, is_downloadable |  
| **ProductOutSearch** |  Schema de produto (busca) |  `fastapi_app/models.py` |  id, title, image, link |  
| **OrderItem** |  Item de pedido |  `fastapi_app/models.py` |  vendor_id, product_id, variation_id, quantity, product_price, purchase_price, item_id, item_type, item_total, item_sub_total, commission_amount, platform_commission, affiliate_commission, vendor_commission, discount_amount, discount_type |  
| **OrderPayload** |  Payload de pedido |  `fastapi_app/models.py` |  order_id, customer_id, payment_method, requires_shipping, shipping, tax, shipping_tax, order_status, affiliate_id, affiliate_hash, shipment_id, rate_id, items, billing, shipping_address |  
| **AddressPayload** |  Payload de endereço |  `fastapi_app/models.py` |  order_id, billing, shipping |  
  
   
**Models (models.py):**  
- `ProductOut`: Model para resposta de produtos (VERIFICADO models.py:7)  
- `ProductOutSearch`: Model para busca de produtos (:23)  
- `OrderItem`: Model para itens de pedido (:30)  
- `OrderPayload`: Model para payload de pedido (:51)  
  - Validação: `check_shipping_if_required` (:71)  
- `AddressPayload`: Model para endereços (:97)  
  
---  
**2.6. Utilities**  
  
|  |  |  |  |  
| - | - | - | - |  
| **Componente** |  **Descrição** |  **Path** |  **Funções Principais** |  
| **Database** |  Utilitários de banco de dados |  `fastapi_app/utils/database.py` |  get_connection() |  
  
   
**Database (database.py):**  
- `get_connection()`: Cria conexão com BD (VERIFICADO linha 8-30)  
  - Suporte: MySQL (pymysql) e PostgreSQL (psycopg2) (VERIFICADO database.py:8-30; o engine usado em qualquer ambiente é NAO VERIFICADO)  
  - Configuração via variáveis de ambiente:  
    - DB_ENGINE: mysql (padrão) ou postgresql  
    - MySQL: MYSQL_HOST, MYSQL_PORT, MYSQL_DB, MYSQL_USER, MYSQL_PASSWORD  
    - PostgreSQL: DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD  
  - Retorna: Conexão pymysql ou psycopg2  
  
---  
**3. Autoload de Classes**  
**Frontend (VERIFICADO frontend/grpms-main.php:1537-1565):**  
- `grpms_autoload()`: Autoload de classes com prefixo `GRPMS_` (`grpms_autoload` :1537)  
- Diretórios escaneados:  
  - `includes/admin/access/classes/`  
  - `includes/admin/platform/classes/`  
  - `includes/general/classes/`  
  - `includes/profile/classes/`  
- Padrão de arquivo: `class-{nome-normalizado}.php`  
- Exemplo: `GRPMS_Access_Validator` → `class-grpms-access-validator.php`  
**Inicialização de classes (frontend/grpms-main.php, bloco por volta de :1575-1590; as linhas :1651-1685 citadas antes caem em bloco comentado — ver nota):**  
- `GRPMS_Access_Capability::ensure()` (VERIFICADO frontend/grpms-main.php:1565; hook `init` prioridade 1; `register_activation_hook` :1566 NAO VERIFICADO nesta leitura)  
- `GRPMS_Gen_Spinner()` (VERIFICADO frontend/grpms-main.php:1576-1577)  
- `GRPMS_TOTP_Login_Gate()` (VERIFICADO frontend/grpms-main.php:1583-1584)  
- `GRPMS_User_Session_Audit()` (VERIFICADO frontend/grpms-main.php:1587-1588)  
- Handlers AJAX (`GRPMS_Access_Create_Ajax` instanciado em :1597, VERIFICADO; linhas dos demais NAO VERIFICADAS):  
  - `GRPMS_Access_Create_Ajax`  
  - `GRPMS_Access_Save_Ajax`  
  - `GRPMS_Access_Delete_Ajax`  
  
---  
**4. UNVERIFIED Items**  
- UNVERIFIED: Lista completa de classes nos diretórios de autoload (necessita verificação em tempo de execução)  
- UNVERIFIED: Comportamento em runtime dos services (assinaturas, docstrings e dependências foram lidas em ARCH-FIX-03; execução não)  
- UNVERIFIED: Parâmetros e contratos de cada rota de dashboard/galeria/mobile (as rotas e autenticação foram listadas em 2.3.1; corpos de resposta não)  
