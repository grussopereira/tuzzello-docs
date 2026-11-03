> **STATUS DO DOCUMENTO: PARCIAL** (revisado em 2026-11-03 após validação ARCH-VALIDATE-01 / ARCH-FIX-01). Citações de linha corrigidas contra o HEAD de frontend e backend; o modelo de implantação descrito é o evidenciado pelas docs de implantação (VMs, não contêineres). Itens não confirmáveis no código estão marcados **NAO VERIFICADO**: Nginx, bloqueio público de rotas, status de produção de qualquer componente, Stripe/WooPayments, PayPal, Podman, YITH Affiliates, topologia/IPs, cron (apenas em README-FE.md), instalação do MU-Plugin de mídia. Flags com padrão desligado (`GRPMS_COMMISSION_WATERFALL`, `EASYPOST_BUY_LIVE`, `GRPMS_AUTO_COMPLETE`, `GRPMS_WC_SEND_ENABLED`) estão descritas como tal.
> **ARCH-FIX-04 (2026-11-03) — Stripe e systemd:** (a) Stripe (ARCH-FIX-05): segundo o Product Owner (informação do proprietário, aceita como declarada), o Stripe é o provedor de pagamento da plataforma, usado por meio de um plugin do WooCommerce. O mecanismo dentro do WordPress (qual plugin, versão e configuração) é **NAO VERIFICADO**: não está nestes repositórios. Nenhuma chamada à API Stripe, PaymentIntent, Checkout Session, webhook Stripe ou verificação de assinatura existe em `frontend/` e `backend/fastapi_app` neste HEAD (apenas fixtures em frontend/tests/payout-approval-guard.php:367,395); o checkout guarda o identificador genérico de método de pagamento do WooCommerce e o backend guarda o `transaction_id` recebido do webhook de pedido do WooCommerce. O ponto de integração é o WooCommerce, não o backend. Referências de configuração fora do código (somente nomes de arquivo e linhas; nenhum conteúdo aberto ou registrado): backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (drop-in `stripe.conf`) e :24 (arquivo `backend/infrastructure/secrets/stripe.env`, referência de arquivo apenas, ausente neste workspace); frontend/docs/REGISTRO-ERROS-CORRECOES.md:579. (b) systemd: não consta em CLOUD-DEPLOYMENT.md (frontend e backend); consta em backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (alegação do repositório lab, conflitante com a decisão do PO `grpms-backend`: serviço de usuário `tuzzello-backend`, **NAO VERIFICADO**) e frontend/docs/REGISTRO-ERROS-CORRECOES.md:579; nome de serviço: decisão do PO = `grpms-backend`; a citação `tuzzello-backend` (PROCESSO-AUTOMATICO-CONCLUSAO.md:22) é alegação do repositório conflitante, NAO VERIFICADO; código/implantação: **NAO VERIFICADO**. (c) Citações de arquivo usam caminho completo a partir da raiz do repositório (`frontend/…`, `backend/…`).
> **ARCH-FIX-06 (2026-11-03) — Nome do serviço:** o nome do serviço de backend foi decidido pelo Product Owner como `grpms-backend` (decisão vinculante). O estado da unidade systemd local é **NAO VERIFICADO** (o barramento de usuário estava inacessível). A alegação do repositório lab `backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22` (serviço de usuário `tuzzello-backend`) conflita com a decisão do PO e permanece citada apenas como alegação: **NAO VERIFICADO**.

> **ARCH-FIX-03 (2026-11-03):** releitura de cada ADR, afirmação por afirmação, contra `frontend/` e `backend/`. Convenção: **VERIFICADO (arquivo:linha)** = confirmado no código; **NAO VERIFICADO** = não confirmado por código. Regra aplicada a todos os ADRs: o campo **Decisão** foi verificado contra o código quando cita código; os campos **Contexto**, **Consequências** e **Alternativas Consideradas** são racionalização do arquiteto, sem fonte nos repositórios, e portanto **NAO VERIFICADOS** (não há registro de decisão original nos repositórios). Cada ADR traz abaixo um bloco "Verificação por afirmação". O documento permanece **PARCIAL**: restam afirmações NAO VERIFICADAS (implantação em VMs, produção, cron, MU-Plugin, GCP).

# Decisões Arquiteturais do Sistema Tuzzello

> **Propósito:** Documentar as Architecture Decision Records (ADRs) do sistema Tuzzello com base no código e documentação existente. **Data:** 2026-11-03. Descreve o código no HEAD dos repositórios. Decisões inferidas estão marcadas como INFERRED.

---

## Formato dos ADRs

Cada ADR segue o formato:
```
## [ID] Título da Decisão

**Status:** Aprovado/Proposto/Rejeitado/Depreciado  
**Contexto:** Descrição do problema ou oportunidade  
**Decisão:** Solução escolhida  
**Consequências:** Impactos positivos e negativos  
**Alternativas Consideradas:** Outras opções avaliadas  
**Fontes:** Arquivos e linhas que comprovam a decisão
**Tipo:** Visível no código / INFERRED
```

---

## 1. Decisões de Estrutura do Sistema

### ADR-001: Separação de Responsabilidades entre WordPress e Backend FastAPI

**Status:** Aprovado  
**Contexto:** 
- WordPress + WooCommerce + WCFM fornecem funcionalidades de e-commerce e multi-vendedor
- Operações complexas (cálculo de frete, validação de afiliados, processamento de pedidos) requerem lógica de negócios especializada
- Necessidade de isolar lógica crítica de segurança do frontend
- Performance: WordPress/PHP pode ser limitado para operações intensivas em I/O ou CPU

**Decisão:** 
- Manter lógica de apresentação e gerenciamento de conteúdo no WordPress
- Extrair lógica de negócios complexa e integrações externas para um backend FastAPI em Python
- WordPress e FastAPI são serviços separados (VMs distintas segundo a documentação de implantação: `lptaf01web01` e `lptaf01pyt01`) que se comunicam via HTTP. A documentação anterior dizia "containers"; isso não é sustentado pelas fontes (backend/podman é um arquivo vazio de 0 bytes)

**Consequências:**
✅ **Positivas:**
- Melhor separação de responsabilidades
- Backend em Python permite uso de bibliotecas especializadas (EasyPost, cálculos financeiros)
- Isolamento de segurança: backend não exposto diretamente ao navegador
- Escalabilidade independente dos serviços

❌ **Negativas:**
- Complexidade aumentada na arquitetura
- Necessidade de sincronização de dados entre WordPress BD e backend
- Latência adicional nas chamadas entre serviços
- Gerenciamento de dois ambientes de execução

**Alternativas Consideradas:**
- Todo o sistema no WordPress (rejeitado: complexidade e performance)
- Todo o sistema no FastAPI (rejeitado: perderia vantagens do WordPress/WooCommerce)
- Microserviços (rejeitado: complexidade excessiva para o escopo atual)

**Fontes:**
- frontend docs/CLOUD-DEPLOYMENT.md:3-4 e backend docs/CLOUD-DEPLOYMENT.md:3 (VMs de destino)
- fastapi_app/main.py (registro de routers; webhook WooCommerce em :24, CORS em :26-32; linhas corrigidas)
- includes/api/grpms-backend-client.php:1-110 (cliente de integração)
- fastapi_app/auth/wordpress_bridge.py:10-15 (autenticação entre serviços)

**Verificação por afirmação (ARCH-FIX-03):**
- Decisão: WordPress com apresentação e backend FastAPI com lógica/integrações: VERIFICADO (grpms-backend-client.php:27; main.py:23-48; routers de checkout, frete, token e dashboard). Comunicação HTTP: VERIFICADO.
- "VMs distintas lptaf01web01/lptaf01pyt01": NAO VERIFICADO no código; fonte: frontend docs/CLOUD-DEPLOYMENT.md:3-4 e backend docs/CLOUD-DEPLOYMENT.md:3 (documentação de implantação).
- Consequência "backend não exposto diretamente ao navegador": **contradita pelo código**: o navegador chama o backend com JWT (apiUrl passado ao JS, frontend/includes/seller/grpms-seller-dashboard.php:171-186; frontend/includes/affiliation/grpms-affiliate-dashboard.php:120) e o CORS permite origem de navegador (main.py:26-32).
- "Escalabilidade independente", "latência adicional", "isolamento de segurança": NAO VERIFICADO (avaliação do arquiteto). Alternativas consideradas: NAO VERIFICADO (sem registro).

**Tipo:** Visível no código

---

### ADR-002: Autenticação via Segredo Compartilhado entre WordPress e Backend

**Status:** Aprovado  
**Contexto:** 
- Necessidade de WordPress autenticar-se no backend para acessar endpoints protegidos
- Segredo não pode ser exposto no frontend (navegador)
- Solução deve ser simples e performática

**Decisão:** 
- Usar um segredo compartilhado (`GRPMS_WP_SHARED_SECRET`) configurado em ambos os ambientes
- WordPress envia header `X-GRPMS-WP-Secret` somente nas chamadas servidor-a-servidor feitas por `grpms_backend_http()` às seis rotas permitidas (grpms-backend-client.php:33-35; header em :46-48). Chamadas de dados dos painéis usam JWT Bearer. Há um segundo mecanismo, de identidade de vendedor assinada por HMAC (`X-GRPMS-Vendor-Id/Ts/Sig`, `GRPMS_WP_IDENTITY_KEY`; wordpress_bridge.py:18-30), usado em rotas de rastreio do EasyPost (easypost_router.py:3, :89, :105, :127)
- Backend valida segredo via `secrets.compare_digest()` (comparação constante-time para evitar timing attacks)
- Segredo nunca é exposto no frontend ou armazenado no Git

**Consequências:**
✅ **Positivas:**
- Simplicidade: não requer infraestrutura complexa (OAuth, certificado mTLS)
- Performance: validação rápida via string comparison
- Segurança: segredo não sai do servidor, comparação constante-time

❌ **Negativas:**
- Risco se segredo for comprometido (acesso total ao backend)
- Todos os serviços WordPress compartilham o mesmo segredo
- Não há granularidade de permissões baseada em segredo

**Alternativas Consideradas:**
- JWT assinado pelo WordPress (rejeitado: complexidade adicional)
- mTLS entre serviços (rejeitado: complexidade de configuração)
- OAuth 2.0 client credentials (rejeitado: overkill para o caso de uso)

**Fontes:**
- fastapi_app/auth/wordpress_bridge.py:10-15 (implementação da validação)
- includes/api/grpms-backend-client.php:27-29, :46-48 (verificação de configuração e envio do header)
- backend/.env.example:7 (configuração do segredo)
- frontend docs/CLOUD-DEPLOYMENT.md:44-48 (constantes no wp-config.php)

**Verificação por afirmação (ARCH-FIX-03):**
- Segredo compartilhado `GRPMS_WP_SHARED_SECRET`, header `X-GRPMS-WP-Secret`, `compare_digest`, mínimo de 32 caracteres: VERIFICADO (wordpress_bridge.py:10-15; grpms-backend-client.php:27-29, :46-48).
- Seis rotas permitidas: VERIFICADO (grpms-backend-client.php:33-35); uma delas, `/api/checkout/save-commissions`, não tem rota no backend: NAO VERIFICADO como rota.
- Identidade de vendedor por HMAC (`GRPMS_WP_IDENTITY_KEY`) nas rotas de rastreio: VERIFICADO (wordpress_bridge.py:18-35; easypost_router.py:3, :89, :105, :127).
- "Segredo nunca é armazenado no Git": NAO VERIFICADO para o histórico do Git; o `.env.example` só lista nomes (VERIFICADO .env.example:7). "Todos os serviços WordPress compartilham o mesmo segredo": NAO VERIFICADO (topologia).
- Contexto/alternativas (JWT, mTLS, OAuth): NAO VERIFICADO.

**Tipo:** Visível no código

---

### ADR-003: Backend Acessa Banco de Dados WordPress Diretamente

**Status:** Aprovado  
**Contexto:** 
- Backend precisa acessar dados do WordPress (usuários, pedidos, produtos)
- WordPress já possui API REST, mas pode não expor todos os dados necessários
- Performance: acesso direto ao BD é mais rápido que via API REST do WordPress
- Backend é confiável e está no mesmo ambiente de rede

**Decisão:** 
- Backend FastAPI conecta diretamente ao MySQL do WordPress
- Usa pymysql (MySQL) ou psycopg2 (PostgreSQL) para conexão
- Credenciais configuradas via variáveis de ambiente
- Backend executa queries SQL diretamente nas tabelas wp_*

**Consequências:**
✅ **Positivas:**
- Performance ótima para queries complexas
- Acesso a todos os dados do WordPress sem restrições da API
- Simplicidade na implementação

❌ **Negativas:**
- Acoplamento forte entre backend e schema do WordPress
- Mudanças no schema WordPress podem quebrar backend
- Risco de segurança se credenciais forem comprometidas
- Violação do princípio de encapsulation

**Alternativas Consideradas:**
- Acessar apenas via WordPress REST API (rejeitado: performance e limitações)
- Replicar dados necessários para BD separado (rejeitado: complexidade de sincronização)
- Usar WordPress como proxy para todas as queries (rejeitado: latência)

**Fontes:**
- fastapi_app/utils/database.py:8-30 (implementação da conexão)
- fastapi_app/routers/token_router.py:24 (query direta em wp_usermeta) e :29 (phpserialize.loads)
- backend/fastapi_app/services/checkout/order_service.py:42-80 (queries em wp_woocommerce_*)

**Verificação por afirmação (ARCH-FIX-03):**
- Backend conecta ao banco do WordPress com `pymysql` (padrão) ou `psycopg2` e lê tabelas `wp_*`: VERIFICADO (database.py:8-30; token_router.py:24; checkout_service.py:9; backend/fastapi_app/services/checkout/order_service.py:50). Credenciais por variáveis de ambiente: VERIFICADO (database.py:13-27); suporte efetivo a PostgreSQL: NAO VERIFICADO (SQL específico de MySQL, ex.: `FIND_IN_SET` shipping_service.py:12).
- Contexto "backend no mesmo ambiente de rede", "performance superior à REST": NAO VERIFICADO. Alternativas: NAO VERIFICADO.

**Tipo:** Visível no código

---

### ADR-004: Uso de JWT para Autenticação de Usuários no Backend

**Status:** Aprovado  
**Contexto:**
- Usuários autenticados no WordPress precisam acessar endpoints protegidos do backend
- Necessidade de validação de identidade e roles do usuário
- Solução deve ser stateless e escalável

**Decisão:** 
- Backend emite tokens JWT (RS256) para usuários autenticados no WordPress
- Token contém: user_id, roles, app (grpms_plugin)
- Tokens são renovados por AJAX no frontend com nonce do WordPress
- Expiração configurável via `JWT_EXPIRE_MINUTES` (padrão: 15 minutos)

**Consequências:**
✅ **Positivas:**
- Stateless: backend não precisa manter sessão
- Escalável: não depende de storage de sessão
- Segurança: tokens assinados não podem ser forjados
- Flexibilidade: pode incluir claims customizados

❌ **Negativas:**
- Tokens não podem ser invalidados antes da expiração (sem lista de revogação)
- Necessidade de renovação periódica
- Complexidade na implementação de refresh tokens

**Alternativas Consideradas:**
- Sessões server-side (rejeitado: não escalável)
- OAuth 2.0 (rejeitado: complexidade excessiva)
- Cookies assinados (rejeitado: menos flexível que JWT)

**Fontes:**
- fastapi_app/auth/jwt_rs256.py (implementação JWT)
- fastapi_app/routers/token_router.py:18-42 (endpoint de geração)
- includes/api/grpms-backend-client.php:83-110 (renovação por AJAX)
- backend/.env.example:8 (configuração de expiração)

**Verificação por afirmação (ARCH-FIX-03):**
- JWT RS256, payload `user_id`/`roles`/`app`, expiração padrão de 15 minutos por `JWT_EXPIRE_MINUTES`: VERIFICADO (jwt_rs256.py:16-17, :19-23; token_router.py:35; .env.example:8).
- Renovação por AJAX com nonce do WordPress: VERIFICADO (grpms-backend-client.php:88-110).
- "Sem lista de revogação": VERIFICADO por leitura de jwt_rs256.py:19-27 e dependencies.py:9-17 (nenhum mecanismo de revogação localizado).
- Contexto "stateless e escalável" e alternativas (sessões, OAuth, cookies): NAO VERIFICADO.

**Tipo:** Visível no código

---

## 2. Decisões de Segurança

### ADR-010: Bloqueio de Acesso Móvel para Áreas de Gestão

**Status:** Aprovado  
**Contexto:** 
- Interface de gestão (dashboard, mydash) é complexa e não otimizada para mobile
- Acesso móvel pode expor funcionalidades sensíveis em dispositivos não seguros
- Necessidade de prevenir problemas de UX e segurança

**Decisão:** 
- Bloquear acesso a `/mydash/*` e `/app/*` para usuários em dispositivos móveis
- Detecção via User-Agent string (frontend/grpms-main.php:42)
- Redirecionar para `/no-mobile-access` com mensagem clara
- Whitelist de paths públicos permitidos (VERIFICADO frontend/grpms-main.php:745-775; citação anterior :500-520 estava incorreta)

**Consequências:**
✅ **Positivas:**
- Melhor UX: evita interface quebrada em mobile
- Segurança: reduz exposição de funcionalidades sensíveis
- Feedback claro para usuário

❌ **Negativas:**
- User-Agent pode ser forjado (mitigação: cookie adicional)
- Whitelist requer manutenção contínua
- Não permite uso legítimo em tablets

**Alternativas Consideradas:**
- Design responsivo para mobile (rejeitado: complexidade e custo)
- Aplicativo mobile nativo (rejeitado: fora do escopo atual)
- PWA otimizada para mobile (rejeitado: ainda em desenvolvimento)

**Fontes:**
- frontend/grpms-main.php:80-127 e :1357-1362 (bloqueio de login mobile)
- frontend/grpms-main.php:727 (`grpms_public_pages_gate`), add_action em :805 (prioridade 9)
- frontend/grpms-main.php:48-52 (detecção de path /app)
- frontend/grpms-main.php:42 (grpms_is_mobile_user_agent)

**Verificação por afirmação (ARCH-FIX-03):**
- Bloqueio de `/mydash/*` e `/app/*` em dispositivos móveis por User-Agent, redirecionamento para `/no-mobile-access`: VERIFICADO (frontend/grpms-main.php:42, :79-81, :113-127, :1357-1362).
- Whitelist de paths públicos: VERIFICADO (frontend/grpms-main.php:745-775).
- "Mitigação: cookie adicional" para User-Agent forjado: NAO VERIFICADO (nenhum cookie de mitigação localizado no gate; existe cookie de intenção de login do PWA, grpms-pwa.php:10-28, com outro propósito).
- "PWA ainda em desenvolvimento" (alternativa rejeitada): NAO VERIFICADO; o módulo PWA existe (grpms-pwa.php:315, :358). Demais contexto/consequências: NAO VERIFICADO.

**Tipo:** Visível no código

---

### ADR-011: Limitação de Acesso à Mídia por Proprietário

**Status:** Proposto em documentação (README); implementação NAO VERIFICADA  
**Contexto:** 
- Vendedores devem ver apenas suas próprias mídias (imagens, vídeos, arquivos)
- Administradores precisam de acesso irrestrito para gerenciamento
- WordPress não fornece controle granular de acesso à mídia por padrão

**Decisão:** 
- Implementar MU-Plugin (`grpms-limit-media-to-owner.php`) para limitar acesso
- Apenas roles `administrator` e `shop_manager` têm acesso irrestrito
- Todos os outros usuários veem apenas mídia com `post_author = current_user_id`
- Aplicar filtros em multiple pontos:
  - Modal de mídia (AJAX): `ajax_query_attachments_args`
  - Listagem de mídia (admin): `pre_get_posts`
  - REST API: `rest_attachment_query`
  - Tela direta de edição: `load-post.php`
  - Consultas SQL gerais: `posts_clauses_all`

**Consequências:**
✅ **Positivas:**
- Segurança: prevenir acesso não autorizado à mídia de outros vendedores
- Isolamento de dados entre vendedores
- Implementação abrangente (cobre todos os casos de uso)

❌ **Negativas:**
- Complexidade na implementação
- Possível impacto em performance (filtros em todas as queries)
- Manutenção de MU-Plugin adicional

**Fontes:**
- README-FE.md:~102-104 em diante (instruções e código do MU-Plugin; o arquivo `wp-content/mu-plugins/grpms-limit-media-to-owner.php` não existe nos repositórios)
- README-FE.md:~102-104 em diante (definição de roles privilegiadas, no texto do MU-Plugin)

**Verificação por afirmação (ARCH-FIX-03):**
- Existência do MU-Plugin `grpms-limit-media-to-owner.php` nos repositórios: NAO VERIFICADO (busca negativa; o nome só ocorre em frontend/README-FE.md). Instalação em qualquer ambiente: NAO VERIFICADO. Os filtros listados (`ajax_query_attachments_args`, `pre_get_posts`, `rest_attachment_query`, `load-post.php`, `posts_clauses_all`) são conteúdo do README: NAO VERIFICADO no código.

**Tipo:** Visível na documentação (README). **Implementação: NAO VERIFICADO** — o MU-Plugin não está nos repositórios; não é possível afirmar que está instalado.

---

### ADR-012: Forçar Produtos Virtuais para Vendedores

**Status:** Aprovado  
**Contexto:** 
- Vendedores (não administradores) só devem cadastrar produtos virtuais
- Produtos físicos requerem logística complexa (frete, estoque, fulfillment)
- Plataforma não está preparada para gerenciar produtos físicos de todos os vendedores

**Decisão:** 
- Forçar `post_meta._virtual = 'yes'` para todos os produtos criados por vendedores
- Aplicar em múltiplos hooks para cobrir todos os casos:
  - `save_post_product`: ao salvar produto (VERIFICADO frontend/grpms-main.php:1298; citação anterior :883-899 incorreta)
  - `wcfm_product_manager_data_filter`: nos dados do WCFM (VERIFICADO frontend/grpms-main.php:1310; anterior :902-907 incorreta)
  - `after_wcfm_products_manage_meta_save`: após salvar meta do WCFM (VERIFICADO frontend/grpms-main.php:1320; anterior :911-912 incorreta)
  - `wcfm_products_manage_meta_save`: ao salvar meta (VERIFICADO frontend/grpms-main.php:1321; anterior :912 incorreta)
- Verificar se usuário é vendedor via `grpms_should_force_vendor_virtual_products()` (frontend/grpms-main.php:1264)
- Exceções: DOING_AJAX, DOING_AUTOSAVE, wp_is_post_revision, wp_is_post_autosave

**Consequências:**
✅ **Positivas:**
- Garante consistência: todos os produtos de vendedores são virtuais
- Simplifica logística da plataforma
- Previne erros de configuração

❌ **Negativas:**
- Limitação para vendedores que querem vender produtos físicos
- Solução temporária até implementação completa de logística
- Puede causar confusão para vendedores

**Fontes:**
- frontend/grpms-main.php:1263-1321 (função em :1264; hooks em :1298, :1310, :1320, :1321)
- frontend/grpms-main.php:1263 (comentário: "Temporariamente, vendedores so podem cadastrar produtos virtuais")

**Verificação por afirmação (ARCH-FIX-03):**
- Função `grpms_should_force_vendor_virtual_products` (:1264), meta `_virtual = yes` (:1270), hooks (:1298, :1310, :1320, :1321): VERIFICADO.
- Exceções: AJAX `save_product_changes`, autosave e revisões: VERIFICADO (frontend/grpms-main.php:1279-1296); a exceção é restrita à ação `save_product_changes`, não a todo `DOING_AJAX` como escrito acima.
- "Verificar se o usuário é vendedor": role `wcfm_vendor` VERIFICADO como string (:1266); a existência do plugin WCFM: NAO VERIFICADO.
- Contexto ("plataforma não preparada para produtos físicos") e consequências: NAO VERIFICADO; o comentário "Temporariamente..." em :1263 é VERIFICADO.

**Tipo:** Visível no código

---

### ADR-013: Proteção de Páginas Públicas com Gatekeeper

**Status:** Aprovado  
**Contexto:** 
- Necessidade de controlar acesso a páginas públicas
- Evitar que usuários acessem URLs diretas sem contexto
- Centralizar lógica de redirecionamento

**Decisão:** 
- Implementar gatekeeper via hook `template_redirect` com prioridade 9
- Exceções técnicas sempre permitidas (admin, REST, AJAX, CRON, etc.)
- Whitelist de paths públicos permitidos
- Redirecionar para `/mydash/` por padrão
- Preservar query string para telemetria

**Consequências:**
✅ **Positivas:**
- Controle centralizado de acesso
- Segurança: evita acesso não autorizado via URLs diretas
- Flexibilidade via whitelist

❌ **Negativas:**
- Manutenção da whitelist
- Possível impacto em SEO (redirecionamentos 302)
- Complexidade em casos edge

**Fontes:**
- frontend/grpms-main.php:727 (grpms_public_pages_gate), add_action em :805
- frontend/grpms-main.php:805 (add_action com prioridade 9; citação anterior :479 estava incorreta)

**Verificação por afirmação (ARCH-FIX-03):**
- Gatekeeper `grpms_public_pages_gate` (:727) em `template_redirect` prioridade 9 (:805): VERIFICADO. Exceções técnicas (admin, REST, AJAX, CRON, customize, embed): VERIFICADO (:728-734). Redirecionamento 302 para `/mydash/` preservando query string: VERIFICADO (frontend/grpms-main.php:786-803).
- Contexto e consequências (SEO, edge cases): NAO VERIFICADO.

**Tipo:** Visível no código

---

### ADR-014: Controle de Acesso Baseado em Funções (Access Control)

**Status:** Aprovado  
**Contexto:** 
- Necessidade de controle granular de acesso a funcionalidades do plugin
- WordPress roles (wcfm_vendor, yith_affiliate) não são suficientes para todas as permissões
- Necessidade de auditabilidade de acessos

**Decisão:** 
- Implementar sistema de Access Control com capabilities customizadas
- Definir funções (function keys) para cada funcionalidade
- Validar acesso via `GRPMS_Access_Validator::check($function_key)`
- Registrar capabilities via `GRPMS_Access_Capability::ensure()`
- Auditar acessos via `GRPMS_Access_Audit`
- Integração com REST API via `grpms_rest_can_access()`

**Consequências:**
✅ **Positivas:**
- Controle granular de acesso
- Auditabilidade de quem acessou o que e quando
- Flexibilidade para definir permissões customizadas

❌ **Negativas:**
- Complexidade na implementação
- Manutenção de capabilities e function keys
- Possível performance impact (validações freqüentes)

**Fontes:**
- includes/api/grpms-rest-api.php:53-70 (grpms_rest_can_access)
- includes/admin/access/grpms-access-control.php (módulo principal)
- includes/admin/access/classes/class-grpms-access-capability.php
- includes/admin/access/classes/class-grpms-access-validator.php
- frontend/grpms-main.php:1565 (GRPMS_Access_Capability::ensure; citação anterior :1651 estava incorreta, a linha é vazia)

**Verificação por afirmação (ARCH-FIX-03):**
- `GRPMS_Access_Validator::check`, `GRPMS_Access_Capability::ensure` (frontend/grpms-main.php:1565), `GRPMS_Access_Audit`, `grpms_rest_can_access` (grpms-rest-api.php:51-74): VERIFICADO. Funções ("function keys") por piso de roles: VERIFICADO (class-grpms-access-floor.php:5-27).
- Contexto "roles wcfm_vendor/yith_affiliate insuficientes": NAO VERIFICADO (plugins externos). Consequências: NAO VERIFICADO.

**Tipo:** Visível no código

---

## 3. Decisões de Negócio

### ADR-020: Suporte a Multi-Moeda

**Status:** Aprovado  
**Contexto:** 
- Vendedores e compradores de diferentes países
- Necessidade de preços em moedas locais
- Comissões e pagamentos em diferentes moedas

**Decisão:** 
- Suporte a moedas: EUR, USD, BRL, ZAR, XOF (frontend/grpms-main.php:808)
- Moeda preferida do usuário armazenada em user_meta
- Funções de normalização e conversão de moedas
- Obtenção de taxas de câmbio via APIs externas (Frankfurter.dev, floatrates.com)
- Cache de taxas por 24 horas
- Atribuição automática de moeda ao produto baseada no vendedor

**Consequências:**
✅ **Positivas:**
- Flexibilidade para vendedores e compradores internacionais
- Preços em moeda local
- Conversão automática de taxas

❌ **Negativas:**
- Complexidade na implementação
- Dependência de APIs externas (pode falhar)
- Cálculos financeiros mais complexos
- UNVERIFIED: Conversão de frete para moeda do checkout (frontend docs/CLOUD-DEPLOYMENT.md:24-25)

**Fontes:**
- frontend/grpms-main.php:807-1123 (funções de moeda; citação anterior :620-750 incorreta)
- frontend/grpms-main.php:1145-1221 (frankfurter/floatrates em :1145-1176; set_transient em :1221)
- frontend/grpms-main.php:1089-1123 (atribuição de moeda ao produto; anterior :1085-1116 imprecisa)
- backend/.env.example:14-15 (flag GRPMS_COMMISSION_WATERFALL: não pertence ao ADR de multi-moeda; citação mantida apenas como referência cruzada ao ADR-021)

**Verificação por afirmação (ARCH-FIX-03):**
- Moedas EUR, USD, BRL, ZAR, XOF (:808), moeda preferida em user_meta (`grpms_get_user_preferred_currency`, usado em frontend/grpms-main.php:1089-1105), cache de 24 h (:1221), atribuição de moeda ao produto (:1089): VERIFICADO.
- Provedores Frankfurter v2/v1 e floatrates: VERIFICADO (:1145-1176). Fallback com última taxa válida e cache de 1 h: VERIFICADO (:1142, ~:1225-1242).
- "Conversão de frete para a moeda do checkout": NAO VERIFICADO (documentação, frontend docs/CLOUD-DEPLOYMENT.md:24-25). Contexto "comissões e pagamentos em diferentes moedas": VERIFICADO em parte (moeda de repasse, grpms-payment.php:9-53).

**Tipo:** Visível no código

---

### ADR-021: Comissões em Cascata

**Status:** Desativado (Configurável)  
**Contexto:** 
- Necessidade de distribuir comissões entreMultiple partes (plataforma, vendedor, afiliado)
- Lógica complexa de cálculo financeiro
- Impacto direto no dinheiro de vendedores e afiliados

**Decisão:** 
- Implementar lógica de comissões em cascata. A ordem real no código é plataforma → afiliado (sobre o restante) → vendedor recebe o restante (VERIFICADO backend/fastapi_app/services/checkout/commission_math.py:8-11); a ordem anterior "vendor → plataforma → afiliado" estava incorreta
- Configuração via variável de ambiente `GRPMS_COMMISSION_WATERFALL`
- Padrão: 0 (desativado)
- Comentário da fonte: "needs PO authorisation" (.env.example:14; frontend docs/CLOUD-DEPLOYMENT.md:38). "PO" não é expandido no código; a expansão anterior "Purchase Order" não é sustentada: NAO VERIFICADO (esclarecer com o Product Owner)

**Consequências:**
✅ **Positivas:**
- Flexibilidade no modelo de negócios
- Pode ser ativado quando necessário
- Lógica já implementada

❌ **Negativas:**
- Complexidade na implementação
- Risco financeiro se configurado incorretamente
- Requer autorização formal para ativação

**Fontes:**
- backend/.env.example:15 (GRPMS_COMMISSION_WATERFALL=0; padrão desligado)
- backend/.env.example:14 (comentário: "default OFF; changes affiliate and vendor money; needs PO authorisation")
- backend/fastapi_app/services/checkout/commission_math.py (lógica de cálculo)

**Verificação por afirmação (ARCH-FIX-03):**
- Flag `GRPMS_COMMISSION_WATERFALL`, padrão desligado, cascata aplicada só quando ativa: VERIFICADO (backend/fastapi_app/services/checkout/order_service.py:133-140, :196-199; .env.example:14-15). Comentário "needs PO authorisation": VERIFICADO (.env.example:14); significado de "PO": NAO VERIFICADO.
- Ordem da cascata: plataforma, depois afiliado sobre o restante, vendedor com o resto: VERIFICADO (backend/fastapi_app/services/checkout/commission_math.py:8-11); a versão anterior do ADR estava incorreta.
- "Requer autorização formal para ativação" como controle técnico: NAO VERIFICADO (é um comentário, não um mecanismo no código).

**Tipo:** Visível no código e documentação

---

### ADR-022: Produtos Virtuais Apenas

**Status:** Aprovado (Temporário)  
**Contexto:** 
- Plataforma não está preparada para logística de produtos físicos
- Frete, estoque e fulfillment complexos
- Foco atual em produtos digitais

**Decisão:** 
- Forçar todos os produtos de vendedores a serem virtuais (ADR-012)
- Frete físico limitado: 1 produto/1 unidade por cotação (frontend docs/CLOUD-DEPLOYMENT.md:22)
- Origem completa do vendedor obrigatória para frete internacional (frontend docs/CLOUD-DEPLOYMENT.md:26)
- Alfândega internacional bloqueada até uso de dados reais do produto (frontend docs/CLOUD-DEPLOYMENT.md:26-27; documentação)

**Consequências:**
✅ **Positivas:**
- Simplificação da logística
- Foco em modelo de negócios claro
- Evita problemas com frete e estoque

❌ **Negativas:**
- Limitação para vendedores de produtos físicos
- Solução temporária
- Pode limitar crescimento do marketplace

**Fontes:**
- frontend/grpms-main.php:1263 (comentário: "Temporariamente, vendedores so podem cadastrar produtos virtuais")
- frontend docs/CLOUD-DEPLOYMENT.md:22-27 (limitações de frete)

**Verificação por afirmação (ARCH-FIX-03):**
- Forçar produtos virtuais (ADR-012): VERIFICADO. Limites de frete físico (1 produto/1 unidade, origem completa, alfândega): fonte é frontend docs/CLOUD-DEPLOYMENT.md:22-27 (documentação). No código, VERIFICADO apenas que a revalidação de afiliado exige um único item no carrinho (grpms-guest-shipping.php:211-215); a regra "1 unidade por cotação" e o bloqueio de alfândega: NAO VERIFICADO.

**Tipo:** Visível no código e documentação

---

## 4. Decisões de Integração

### ADR-030: Integração com EasyPost para Frete

**Status:** Homologação  
**Contexto:** 
- Necessidade de cotação de frete para produtos físicos
- Suporte a frete internacional
- Integração com transportadoras

**Decisão:** 
- Usar EasyPost como provedor de frete
- Modo de operação: teste (EASYPOST_MODE=test) ou produção
- Chaves separadas para teste e produção
- Compra de etiquetas desativada por padrão (EASYPOST_BUY_LIVE=0)
- Compra de etiquetas e webhooks EasyPost desativados na branch de homologação (503: easypost_router.py:85, :192). O webhook assinado de pedidos WooCommerce é um endpoint distinto, padrão desligado (ver ADR-070)

**Consequências:**
✅ **Positivas:**
- Solução profissional para frete
- Suporte a transportadoras internacionais
- API bem documentada

❌ **Negativas:**
- Dependência de serviço externo
- Custo adicional (API key paga em produção)
- Complexidade na integração
- Homologação ainda em andamento

**Fontes:**
- backend/.env.example:10-13 (configurações EasyPost)
- fastapi_app/routers/checkout/easypost_router.py (integração)
- fastapi_app/services/checkout/easypost_service.py (serviço)
- frontend docs/CLOUD-DEPLOYMENT.md:28-29 (etiquetas e webhooks desativados no backend)

**Verificação por afirmação (ARCH-FIX-03):**
- EasyPost como provedor, SDK `easypost.EasyPostClient`, modo por `EASYPOST_MODE`, chaves de teste/produção, `EASYPOST_BUY_LIVE`: VERIFICADO (easypost_service.py:10, :43-45; .env.example:10-13, com `EASYPOST_MODE=test` e `EASYPOST_BUY_LIVE=0` nos exemplos).
- Compra de etiqueta e webhook EasyPost retornam 503: VERIFICADO (easypost_router.py:83-85, :190-192).
- "Status Homologação" e "Custo adicional (API key paga)": NAO VERIFICADO (documentação / avaliação).

**Tipo:** Visível no código e documentação

---

### ADR-031: Rotação de Ownership de Diretórios

**Status:** Aprovado  
**Contexto:** 
- Servidor executando com usuário www-data:www-data
- Arquivos podem ter ownership incorreto após updates ou deployments
- Necessidade de garantir permissões corretas

**Decisão:** 
- Configurar cron jobs para rotacionar ownership de diretórios
- Dois cron jobs:
  - Dia 01 de cada mês, a cada 30 minutos: `*/30 * 1 * * *`
  - Dias 02-31, às 00:10: `10 0 2-31 * *`
- Comando: `/usr/bin/chown -R www-data:www-data /var/www/html/tuzzello/`

**Consequências:**
✅ **Positivas:**
- Garante permissões corretas de arquivos
- Previne problemas de access denied
- Automação da manutenção

❌ **Negativas:**
- Overhead de execução
- Possível impacto em performance durante execução
- Não resolve problemas de permissões em tempo real

**Fontes:**
- README-FE.md:228-249 (crontab para o server; só descrito no README, NAO VERIFICADO)

**Verificação por afirmação (ARCH-FIX-03):**
- NAO VERIFICADO: cron jobs, comandos `chown` e horários constam apenas de README-FE.md:228-249; nenhuma configuração de cron nos repositórios (o `GRPMS_Gen_Shipping` agenda evento WordPress diário para status de frete, class-grpms-gen-shipping.php:64, mas não é este cron). Usuário `www-data`: NAO VERIFICADO.

**Tipo:** Visível na documentação

---

### ADR-032: Validação de URLs da API

**Status:** Aprovado  
**Contexto:** 
- Frontend não deve chamar endpoints arbitrários do backend
- Necessidade de prevenir chamadas não autorizadas
- Segurança: evitar que frontend seja usado como proxy

**Decisão:** 
- Implementar whitelist de rotas permitidas no frontend
- Apenas as seguintes rotas podem ser chamadas:
  - `/api/token/from-wp`
  - `/api/easypost/calculate-shipping`
  - `/api/easypost/validate-selection`
  - `/api/checkout/validate-affiliate`
  - `/api/checkout/save-order`
  - `/api/checkout/save-commissions`
- Qualquer outra rota resulta em erro `grpms_backend_route`

**Consequências:**
✅ **Positivas:**
- Segurança: prevenir chamadas não autorizadas
- Clareza: lista explícita de rotas permitidas
- Facilidade de auditoria

❌ **Negativas:**
- Manutenção da whitelist
- Adicionar nova rota requer atualização em dois lugares (frontend e backend)

**Fontes:**
- includes/api/grpms-backend-client.php:33-35 (whitelist)
- includes/api/grpms-backend-client.php:36-39 (validação `grpms_backend_route`; anterior :31-33 incorreta)

**Verificação por afirmação (ARCH-FIX-03):**
- Whitelist de seis rotas e erro `grpms_backend_route`: VERIFICADO (grpms-backend-client.php:33-39). A whitelist restringe só o cliente PHP; o backend não impõe a lista. `/api/checkout/save-commissions` está na lista mas não existe rota backend: NAO VERIFICADO como rota.
- "Evitar que o frontend seja usado como proxy": VERIFICADO como comentário de intenção (grpms-backend-client.php:32). Consequências: NAO VERIFICADO.

**Tipo:** Visível no código

---

## 5. Decisões de Implementação

### ADR-040: Uso de Pydantic para Validação de Dados

**Status:** Aprovado  
**Contexto:** 
- Necessidade de validação robusta de dados de entrada
- FastAPI já tem suporte nativo a Pydantic
- Tipos de dados complexos e aninhados

**Decisão:** 
- Usar Pydantic models para validação das requisições com corpo ("todas" é impreciso: rotas de query, como validate-affiliate, usam `Query`, checkout_router.py:29-30, e o webhook lê corpo bruto)
- Definir schemas para:
  - Produtos (ProductOut, ProductOutSearch)
  - Pedidos (OrderPayload, OrderItem)
  - Endereços (AddressPayload)
  - Tokens (WPTokenRequest)
- Validação automática pelo FastAPI
- Validação customizada via `@model_validator`

**Consequências:**
✅ **Positivas:**
- Validação robusta e tipada
- Documentação automática via OpenAPI/Swagger
- Reutilização de schemas
- Type hints nativos do Python

❌ **Negativas:**
- Overhead de definição de schemas
- Performance (embora mínimo)
- Curva de aprendizado para Pydantic

**Fontes:**
- fastapi_app/models.py:1-99 (todos os schemas)
- fastapi_app/routers/token_router.py:10-12 (WPTokenRequest)
- fastapi_app/routers/checkout/checkout_router.py:21-25 (AffiliateValidationResponse; anterior :13-23 incorreta)

**Verificação por afirmação (ARCH-FIX-03):**
- Modelos `ProductOut`, `ProductOutSearch`, `OrderItem`, `OrderPayload`, `AddressPayload`, `WPTokenRequest`, validador `check_shipping_if_required`: VERIFICADO (models.py:7, :23, :30, :51, :97; models.py:70-71; token_router.py:10).
- "Todas as requisições": impreciso (rotas de query usam `Query`; webhook usa corpo bruto). Consequências: NAO VERIFICADO.

**Tipo:** Visível no código

---

### ADR-041: Uso de phpserialize para Parse de Capabilities

**Status:** Aprovado  
**Contexto:** 
- WordPress armazena capabilities do usuário serializados com `maybe_serialize()`
- Backend precisa parsear esses dados para obter roles do usuário
- Necessidade de biblioteca compatível com formato do WordPress

**Decisão:** 
- Usar biblioteca `phpserialize` para parsear capabilities
- Chamar `loads(serialized.encode(), decode_strings=True)`
- Obter lista de roles a partir do dicionário capabilities

**Consequências:**
✅ **Positivas:**
- Compatibilidade com formato do WordPress
- Parse correto de dados serializados
- Solução simples e direta

❌ **Negativas:**
- Dependência adicional (phpserialize)
- Possível incompatibilidade com versões futuras do WordPress

**Fontes:**
- fastapi_app/routers/token_router.py:3 (import phpserialize)
- fastapi_app/routers/token_router.py:29 (parse: `loads(serialized.encode(), decode_strings=True)`)

**Verificação por afirmação (ARCH-FIX-03):**
- Uso de `phpserialize.loads(..., decode_strings=True)`: VERIFICADO (token_router.py:3, :29; também checkout_service.py:3, :20). Consequências: NAO VERIFICADO.

**Tipo:** Visível no código

---

### ADR-042: Cache de Taxas de Câmbio

**Status:** Aprovado  
**Contexto:** 
- APIs de taxa de câmbio têm rate limits
- Taxas não mudam freqüentemente
- Necessidade de reduzir chamadas externas

**Decisão:** 
- Implementar cache de taxas de câmbio
- Cache key: `grpms_fx_{currency}_usd_{date}`
- Duração: 24 horas (DAY_IN_SECONDS)
- Fallback: usar última taxa válida (cache de 1 hora) se todas as APIs falharem
- Log de erros para debug

**Consequências:**
✅ **Positivas:**
- Reduz chamadas a APIs externas
- Melhora performance
- Resiliência a falhas (fallback)

❌ **Negativas:**
- Taxas podem estar desatualizadas (até 24 horas)
- Complexidade na implementação de cache
- Manutenção de múltiplos providers

**Fontes:**
- frontend/grpms-main.php:1145-1221 (taxa de câmbio; função exata: NAO VERIFICADA nesta revisão)
- frontend/grpms-main.php:1136-1138 (cache via get_transient; anterior :989-994 incorreta)
- frontend/grpms-main.php:1142 e fallback ~:1225-1242 (última taxa válida em opção `grpms_fx_last_valid_*`; anterior :1038-1044 incorreta)

**Verificação por afirmação (ARCH-FIX-03):**
- Chave `grpms_fx_{moeda}_usd_{data}`, transient de 24 h (DAY_IN_SECONDS), fallback com última taxa válida (opção `grpms_fx_last_valid_*`) cacheado por 1 h, log de erro: VERIFICADO (frontend/grpms-main.php:1136-1142, :1221, ~:1225-1242).
- "APIs de câmbio têm rate limits": NAO VERIFICADO.

**Tipo:** Visível no código

---

### ADR-043: Reconciliação de Pedidos

**Status:** Aprovado  
**Contexto:** 
- Dados de pedidos podem ser inconsistentes entre frontend e backend
- Necessidade de validar integridade dos dados antes do processamento
- Prevenir fraudes e erros de processamento

**Decisão:** 
- Implementar validação de itens do pedido no backend
- Comparar dados da requisição com dados no BD WordPress
- Verificar: product_id, quantity, line_total
- Lançar erro de reconciliação se divergências forem encontradas (o código lança `RuntimeError` em `canonical_items`, backend/fastapi_app/services/checkout/order_service.py:84; `OrderReconciliationError` é a classe de :28 tratada em backend/fastapi_app/routers/checkout/order_router.py:15)
- Log detalhado de reconciliações (backend/fastapi_app/services/checkout/order_service.py:20-23)

**Consequências:**
✅ **Positivas:**
- Previne processamento de pedidos inválidos
- Detecção de fraudes (valores forjados)
- Dados consistentes entre sistemas

❌ **Negativas:**
- Overhead de validação
- Possível falha no processamento de pedidos legítimos
- Complexidade na implementação

**Fontes:**
- backend/fastapi_app/services/checkout/order_service.py:23 (reconciliation function; anterior :20-23)
- backend/fastapi_app/services/checkout/order_service.py:28 (OrderReconciliationError; anterior :26-27)
- backend/fastapi_app/services/checkout/order_service.py:42-84 (canonical_items function; anterior :42-80)

**Verificação por afirmação (ARCH-FIX-03):**
- Comparação de itens do pedido com `wp_woocommerce_order_items`/`itemmeta` (product_id, quantidade, `_line_total`), registro de reconciliação e falha: VERIFICADO (backend/fastapi_app/services/checkout/order_service.py:23, :42-84); HTTP 409 em `OrderReconciliationError`: VERIFICADO (backend/fastapi_app/routers/checkout/order_router.py:15-16).
- "Detecção de fraudes": NAO VERIFICADO (finalidade declarada; o código valida integridade de itens).

**Tipo:** Visível no código

---

## 6. Decisões de Infraestrutura

### ADR-050: Uso de Podman para Containerização

**Status:** NAO VERIFICADO — `backend/podman` é um arquivo vazio (0 bytes), não um diretório, e backend/docs/CLOUD-DEPLOYMENT.md descreve VMs (serviço `grpms-backend`, venv, rsync; :42, :54-59) sem mencionar systemd; `systemd` NÃO consta em backend/docs/CLOUD-DEPLOYMENT.md nem em frontend/docs/CLOUD-DEPLOYMENT.md; consta em backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (alegação do repositório lab, conflitante com a decisão do PO `grpms-backend`: serviço de usuário `tuzzello-backend`, **NAO VERIFICADO**) e é citado em frontend/docs/REGISTRO-ERROS-CORRECOES.md:579 (EnvironmentFile do host). Discrepância de nome de serviço: `grpms-backend` (CLOUD-DEPLOYMENT.md:54, :59) vs `tuzzello-backend` (PROCESSO-AUTOMATICO-CONCLUSAO.md:22, alegação conflitante, NAO VERIFICADO); decisão do PO: `grpms-backend`. Uso de systemd no código/implantação: NAO VERIFICADO). Este ADR **não** deve ser lido como fato de implantação.  
**Contexto:** 
- Necessidade de ambiente consistente entre desenvolvimento e produção
- Facilidade de deployment
- Isolamento de dependências

**Decisão:** 
- Usar Podman para containerização
- Scripts de deployment inclusos no repositório
- Ambiente de desenvolvimento local via containers

**Consequências:**
✅ **Positivas:**
- Ambiente consistente
- Fácil deployment
- Isolamento de dependências

❌ **Negativas:**
- Curva de aprendizado para Podman
- Overhead de containers em desenvolvimento

**Fontes:**
- backend/podman (arquivo de 0 bytes, **vazio**; não é um diretório)
- backend docs/CLOUD-DEPLOYMENT.md (modelo descrito: VM `lptaf01pyt01`, serviço `grpms-backend` :54, :59, venv :42, rsync :55; "systemd": `systemd` NÃO consta em backend/docs/CLOUD-DEPLOYMENT.md nem em frontend/docs/CLOUD-DEPLOYMENT.md; consta em backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (alegação do repositório lab, conflitante com a decisão do PO `grpms-backend`: serviço de usuário `tuzzello-backend`, **NAO VERIFICADO**) e é citado em frontend/docs/REGISTRO-ERROS-CORRECOES.md:579 (EnvironmentFile do host). Discrepância de nome de serviço: `grpms-backend` (CLOUD-DEPLOYMENT.md:54, :59) vs `tuzzello-backend` (PROCESSO-AUTOMATICO-CONCLUSAO.md:22, alegação conflitante, NAO VERIFICADO); decisão do PO: `grpms-backend`. Uso de systemd no código/implantação: NAO VERIFICADO)

**Verificação por afirmação (ARCH-FIX-03):**
- `backend/podman`: arquivo de 0 bytes, VERIFICADO. Qualquer uso de Podman/containers: NAO VERIFICADO. Os campos Decisão/Contexto/Consequências deste ADR não têm sustentação.

**Tipo:** INFERRED e não sustentado pelas fontes (`backend/podman` é um arquivo vazio de 0 bytes)

---

### ADR-051: GCP como Provedor de Cloud

**Status:** Aprovado  
**Contexto:** 
- Necessidade de infraestrutura escalável e confiável
- Integração com serviços GCP (Cloud SQL, Load Balancer, etc.)
- Ambiente de homologação antes de produção

**Decisão:** 
- Usar Google Cloud Platform para hospedagem
- VMs: lptaf01web01 (web) e lptaf01pyt01 (backend)
- Branch de homologação: cloud/secure-integration
- Ambiente novo (novo.tuzzello.com) separado do antigo

**Consequências:**
✅ **Positivas:**
- Infraestrutura escalável e confiável
- Integração nativa com serviços GCP
- Ambiente de homologação isolado

❌ **Negativas:**
- Custo de infraestrutura
- Complexidade de gerenciamento
- Migração do ambiente antigo

**Fontes:**
- frontend docs/CLOUD-DEPLOYMENT.md:3-4 (VM lptaf01web01)
- backend docs/CLOUD-DEPLOYMENT.md:3 (VM lptaf01pyt01)
- frontend docs/CLOUD-DEPLOYMENT.md:44-48 (constantes para a VM nova)
- README-FE.md:12 (Linux Debian/Red Hat/Suse/Arch)

**Verificação por afirmação (ARCH-FIX-03):**
- GCP, VMs `lptaf01web01`/`lptaf01pyt01`, branch `cloud/secure-integration`, domínio novo: fontes são frontend docs/CLOUD-DEPLOYMENT.md:1-4 e backend docs/CLOUD-DEPLOYMENT.md:1-3 (documentação). **NAO VERIFICADO no código**. Cloud SQL, Load Balancer e custos: NAO VERIFICADO (sem fonte). O padrão de CORS no código aponta um domínio `novo.` (main.py:28), VERIFICADO apenas como valor padrão.

**Tipo:** Visível na documentação

---

### ADR-052: PHP 8.4 na VM Nova (homologação)

**Status:** Requisito de homologação; produção NAO VERIFICADA (README-FE.md:249 recarrega `php8.2-fpm`, indicando outro runtime no ambiente descrito pelo README)  
**Contexto:** 
- WordPress e plugins compatíveis com PHP 8.4
- Melhor performance e segurança
- Recursos modernos da linguagem

**Decisão:** 
- Validar PHP 8.4 na VM nova após instalar o plugin (frontend docs/CLOUD-DEPLOYMENT.md:82); a documentação alvo é apenas a VM nova
- Validar compatibilidade antes de deployment

**Consequências:**
✅ **Positivas:**
- Melhor performance
- Segurança aprimorada
- Recursos modernos (typed properties, etc.)

❌ **Negativas:**
- Possível incompatibilidade com plugins antigos
- Requer validação de todos os plugins

**Fontes:**
- frontend docs/CLOUD-DEPLOYMENT.md:82 ("validar PHP 8.4")

**Verificação por afirmação (ARCH-FIX-03):**
- PHP 8.4 como requisito de validação da VM nova: fonte frontend docs/CLOUD-DEPLOYMENT.md:82 (documentação; NAO VERIFICADO no código). README-FE.md:249 recarrega `php8.2-fpm`: documentação divergente, NAO VERIFICADO qual é o runtime.

**Tipo:** Visível na documentação

---

### ADR-070: Webhook Assinado de Pedidos WooCommerce, Outbox e Auto-Complete (padrão desligado)

**Status:** Implementado no código, **desligado por padrão**; ativação/uso em qualquer ambiente NAO VERIFICADO  
**Contexto:** 
- Conclusão automática de pedidos após pagamento confirmado envolve dinheiro (alto risco)

**Decisão (como evidenciada pelo código):** 
- `POST /webhooks/woocommerce/order` (routers/webhooks/woocommerce_order.py; registrado em main.py:24) só responde se `GRPMS_AUTO_COMPLETE` estiver ativo (wc_webhook.py:21-22); caso contrário retorna 404
- Assinatura HMAC-SHA256/base64 com `GRPMS_WC_WEBHOOK_SECRET` (wc_webhook.py:55-70); tópico aceito: `order.updated` (:78)
- `wc_outbox_worker.py` ("No scheduler is installed") só envia ao WooCommerce com `GRPMS_WC_SEND_ENABLED`, `GRPMS_AUTO_COMPLETE` e `GRPMS_ENV=lab` (:128-130)
- `auto_complete.py` descrito como "Unwired, default-off completion of a confirmed payment"

**Consequências:** o estado "webhooks desativados" da documentação de implantação refere-se aos webhooks EasyPost; este mecanismo é separado.

**Verificação por afirmação (ARCH-FIX-03):**
- Endpoint `POST /webhooks/woocommerce/order` registrado em main.py:24, responde só com `GRPMS_AUTO_COMPLETE` ativo (wc_webhook.py:21-22), HMAC-SHA256/base64 com `GRPMS_WC_WEBHOOK_SECRET` (wc_webhook.py:52-70), tópico `order.updated` (:78), limite de corpo 1 MiB (woocommerce_order.py:10): VERIFICADO.
- Outbox só escreve com `GRPMS_WC_SEND_ENABLED`, `GRPMS_AUTO_COMPLETE` e `GRPMS_ENV=lab`: VERIFICADO (wc_outbox_worker.py:128-130); `auto_complete.py` "Unwired, default-off": VERIFICADO (:1). Estado de ativação em qualquer ambiente: NAO VERIFICADO.

**Tipo:** Visível no código

---

## 7. Decisões Temporárias

### ADR-060: EasyPost em Modo Teste

**Status:** Temporário  
**Contexto:** 
- Homologação em andamento
- Ambiente de produção não pronto
- Evitar custos com chamadas de API em produção

**Decisão:** 
- Configurar `EASYPOST_MODE=test` por padrão
- Usar `EASYPOST_TEST_KEY` para desenvolvimento
- Desativar compra de etiquetas (`EASYPOST_BUY_LIVE=0`)

**Consequências:**
- Frete não disponível para uso real
- Necessário configs para produção antes do go-live

**Fontes:**
- backend/.env.example:10-13

**Verificação por afirmação (ARCH-FIX-03):**
- `EASYPOST_MODE=test` e `EASYPOST_BUY_LIVE=0` no exemplo de configuração: VERIFICADO (.env.example:10, :13); "padrão" em runtime: NAO VERIFICADO (depende do `.env` real, que não foi lido). Evitar custos de produção: NAO VERIFICADO (motivação).

**Tipo:** Visível no código

---

### ADR-061: Comissões em Cascata Desativadas

**Status:** Temporário  
**Contexto:** 
- Lógica implementada mas não testada em produção
- Impacto financeiro significativo
- Requer autorização formal

**Decisão:** 
- Manter `GRPMS_COMMISSION_WATERFALL=0` por padrão
- Requer autorização ("needs PO authorisation"; significado de "PO" NAO VERIFICADO)

**Consequências:**
- Modelo de comissões simplificado atual
- Ativação futura quando aprovado

**Fontes:**
- backend/.env.example:14-15

**Verificação por afirmação (ARCH-FIX-03):**
- `GRPMS_COMMISSION_WATERFALL=0` no exemplo e padrão "0" no código: VERIFICADO (.env.example:15; backend/fastapi_app/services/checkout/order_service.py:136). "Lógica não testada em produção": NAO VERIFICADO.

**Tipo:** Visível no código

---

## 8. UNVERIFIED Items

- UNVERIFIED: Detalhes completos da implementação de autoload de classes (necessita execução para verificação)
- UNVERIFIED: Lista completa de capabilities customizadas definidas pelo sistema
- UNVERIFIED: Comportamento em runtime de cada service (assinaturas, docstrings e dependências verificadas em ARCH-FIX-03)
- NAO VERIFICADO: uso de Podman (`backend/podman` é um arquivo vazio de 0 bytes). Modelo de implantação por VMs (`lptaf01web01`, `lptaf01pyt01`) e serviço `grpms-backend`: fonte exclusiva são os documentos de implantação; **NAO VERIFICADO no código**; "systemd": `systemd` NÃO consta em backend/docs/CLOUD-DEPLOYMENT.md nem em frontend/docs/CLOUD-DEPLOYMENT.md; consta em backend/docs/PROCESSO-AUTOMATICO-CONCLUSAO.md:22 (alegação do repositório lab, conflitante com a decisão do PO `grpms-backend`: serviço de usuário `tuzzello-backend`, **NAO VERIFICADO**) e é citado em frontend/docs/REGISTRO-ERROS-CORRECOES.md:579 (EnvironmentFile do host). Discrepância de nome de serviço: `grpms-backend` (CLOUD-DEPLOYMENT.md:54, :59) vs `tuzzello-backend` (PROCESSO-AUTOMATICO-CONCLUSAO.md:22, alegação conflitante, NAO VERIFICADO); decisão do PO: `grpms-backend`. Uso de systemd no código/implantação: NAO VERIFICADO
- UNVERIFIED: Detalhes da configuração do Nginx em produção
- UNVERIFIED: Status exato de cada endpoint em produção
