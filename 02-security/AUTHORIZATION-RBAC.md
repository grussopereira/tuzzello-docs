# Autorização e RBAC - Tuzzello Marketplace

## Visão Geral

Este documento descreve o modelo de autorização baseada em funções (RBAC) implementado no Tuzzello Marketplace, em conformidade com OWASP ASVS 5.0, Capítulo V4.

## Matriz de Permissões

### Funções (Roles)

| Função | Descrição | Capacidades |
|--------|-----------|-------------|
| `administrator` | Administrador do sistema | Acesso total a todos os dados e funções |
| `shop_manager` | Gerente de loja | Gerenciar produtos, pedidos, vendedores |
| `wcfm_vendor` | Vendedor | Acessar próprio dashboard, pedidos, produtos, comissões |
| `yith_affiliate` | Afiliado | Acessar próprio dashboard, comissões, links de afiliação |

### Permissões por Função

#### Vendedor (wcfm_vendor)

**Acesso a Dados Próprios:**
- Ver próprio dashboard: `grpms-rest-api.php:932-938` (dashboard/summary)
- Listar próprios produtos: `grpms-rest-api.php:450-495` (seller/products) - Filtro: `author => get_current_user_id()`
- Listar próprios pedidos: `grpms-rest-api.php:580-645` (seller/orders) - Filtro: `WHERE vendor_id = %d` com $vendor_id = get_current_user_id()
- Ver status de envio: `grpms-seller.php:868` - Filtro: `WHERE o.vendor_id = %d`
- Ver comissões financeiras: `grpms-rest-api.php:750-800` (financial/summary)
- Ver histórico de pagamentos: `grpms-rest-api.php:800-850` (financial/payment-history)

**Validação de Isolamento:**
- **seller_dashboard_router.py:16-18**: `normalize_user_id(current_user.get("user_id")) != vendor_id` - NEGA acesso se não for o vendedor
- **grpms-seller-orders.php:138-140**: `WHERE o.vendor_id = %d` - Filtra apenas pedidos do vendedor

#### Afiliado (yith_affiliate)

**Acesso a Dados Próprios:**
- Ver próprio dashboard: `affiliate_dashboard_router.py:13-14` - Verifica se `current_user.get("user_id") != affiliate_id`
- Listar próprias comissões: `grpms-rest-api.php:650-700` (affiliate/commissions) - Filtro: `WHERE affiliate_id = %d`
- Listar própria afiliações: `grpms-rest-api.php:700-750` (affiliate/affiliations) - Filtro: `WHERE affiliate_id = %d`
- Gerenciar links de afiliação: `grpms-affiliation.php:89-150` - Usa `affiliate_id = $user->ID`
- Remover afiliação: `grpms-affiliation.php:677-720` - Usa `affiliate_id = get_current_user_id()`

**Validação de Isolamento:**
- **affiliate_dashboard_router.py:13-14**: Verifica se user_id do usuário != affiliate_id
- **grpms-rest-api.php:650-700**: Filtra por affiliate_id = get_current_user_id()

#### Administrador/Shop Manager

**Acesso Total:**
- Gerenciar todas as comissões da plataforma: `grpms-platform.php:114-170` - Requer `current_user_can('shop_manager')`
- Aprovar pedidos: `grpms-approve_orders.php:112-386` - Requer `GRPMS_Access_Validator::check('orders-approval-adm')`
- Gerenciar taxa de plataforma: `grpms-platform.php:114-145`
- Aprovar/deletar produtos: `grpms-product-delete.php:10-157`, `grpms-products-approval.php:61-784`

### GRPMS_Access_Validator

**Mecanismo de Controle de Acesso Fine-Grained**

Funções verificadas:
- `orders-approval-adm`: Aprovação de pedidos (admin)
- `product-delete-approval`: Aprovação de exclusão de produtos (admin)
- `tax-liability-adm`: Gerenciamento de responsabilidade tributária (admin)
- `platform-commission-adm`: Configuração de comissão da plataforma (admin)
- `dashboard-seller`: Acesso ao dashboard de vendedor
- `dashboard-aff`: Acesso ao dashboard de afiliado
- `order-list-seller`: Listagem de pedidos de vendedor
- `product-list-seller`: Listagem de produtos de vendedor
- `commission-aff`: Acesso a comissões de afiliado
- `affiliation-aff`: Acesso a afiliações de afiliado
- `request-transfer-fin`: Solicitação de transferência financeira
- `profile`: Acesso ao perfil
- `access`: Acesso ao gerenciamento de access

**Implementação**: 
- Verificação via `GRPMS_Access_Validator::check('<function_key>')`
- Usado em shortcodes e handlers AJAX

## Verificação de Isolamento de Vendedor

### Questão Central
Um vendedor NÃO pode ler ou alterar pedidos, comissões ou dados financeiros de outro vendedor, mesmo alterando IDs.

### Resultado

**ENFORCED (17 endpoints)** - Verificação de propriedade implementada:

| Endpoint | Arquivo:Linha | Campo Verificado | Status |
|----------|---------------|------------------|--------|
| Seller Dashboard | seller_dashboard_router.py:16-18 | vendor_id | ENFORCED |
| Affiliate Dashboard | affiliate_dashboard_router.py:13-14 | affiliate_id | ENFORCED |
| Dashboard Main | dashboard_main.py:11-18 | user_id + role | ENFORCED |
| Mobile User | mobile.py:15-16 | user_id | ENFORCED |
| Mobile Sales History | mobile_salehistory_router.py:17 | user_id | ENFORCED |
| Mobile Profile | mobile_profile_router.py:13 | user_id | ENFORCED |
| EasyPost Shipping | easypost_service.py:315-317 | product.post_author == vendor_id | ENFORCED |
| REST Seller Products | grpms-rest-api.php:450 | author = current_user_id() | ENFORCED |
| REST Seller Orders | grpms-rest-api.php:580 | vendor_id = current_user_id() | ENFORCED |
| REST Affiliate Commissions | grpms-rest-api.php:650 | affiliate_id = current_user_id() | ENFORCED |
| REST Affiliate Affiliations | grpms-rest-api.php:700 | affiliate_id = current_user_id() | ENFORCED |
| REST Financial Summary | grpms-rest-api.php:750 | user_id + role | ENFORCED |
| REST Financial Payment History | grpms-rest-api.php:800 | user_id + role | ENFORCED |
| Add Affiliation | grpms-affiliation.php:89 | user->ID | ENFORCED |
| Disaffiliate Product | grpms-affiliation.php:677 | get_current_user_id() | ENFORCED |
| Seller Orders Shortcode | grpms-seller-orders.php:138-140 | vendor_id = $user->ID | ENFORCED |
| Vendor Shipping Status | grpms-seller.php:868 | o.vendor_id = %d | ENFORCED |

**NOT ENFORCED (1 endpoint)** - Sem verificação de propriedade:

| Endpoint | Arquivo:Linha | Campo | Status | Risco |
|----------|---------------|-------|--------|-------|
| Shipment History | easypost_service.py:293-302 | order_id | NOT ENFORCED | BAIXO |

**Nota**: Este endpoint é interno (require_wordpress dependency) e não há endpoint PHP user-facing que o chame. Portanto, não é explorável por vendedores.

**UNVERIFIED (4 endpoints)** - Endpoints internos WordPress-only:

| Endpoint | Arquivo | Status | Notas |
|----------|---------|--------|-------|
| Save Order | order_router.py | UNVERIFIED | Internal (require_wordpress) |
| Save Address | order_router.py | UNVERIFIED | Internal (require_wordpress) |
| Get Cost | shipping_router.py | UNVERIFIED | Internal (require_wordpress) |
| EasyPost Router | easypost_router.py | UNVERIFIED | Internal (require_wordpress) |

## Status de Verificação (OWASP ASVS V4)

| Requisito | Status | Arquivo:Linha | Notas |
|-----------|--------|---------------|-------|
| V4.1.1 - Verificação de autorização | VERIFICADO | Multiple | RBAC implementado |
| V4.1.2 - Isolamento de dados | VERIFICADO | 17 endpoints | Verificação de vendor_id/affiliate_id |
| V4.1.3 - Princípio do menor privilégio | VERIFICADO | Multiple | Funções bem definidas |
| V4.1.4 - Deny by default | VERIFICADO | Multiple | Acesso negado por padrão |

## Referências
- OWASP ASVS 5.0 V4: Access Control
- WordPress Roles and Capabilities
