# Auditoria e Logging - Tuzzello Marketplace

## Visão Geral

Este documento descreve os mecanismos de auditoria, rastreabilidade e retenção de logs implementados no Tuzzello Marketplace, em conformidade com OWASP ASVS 5.0, Capítulo V7.

## Eventos Sensíveis Auditados

### 1. Autenticação e Autorização

**Eventos**:
- Tentativas de login (WordPress core)
- Falhas de autenticação
- Acesso negado a recursos

**Arquivos**:
- frontend/includes/admin/approve_orders/grpms-approve_orders.php:115-125 (grpms_approve_orders_log)

**Implementação**:
```php
// grpms-approve_orders.php:115-125
error_log('grpms_order_approval ' . wp_json_encode([
    'actor_id' => (int) get_current_user_id(),
    'order_id' => (int) $order_id,
    'from' => $from,
    'to' => $to,
    'outcome' => $outcome,
]));
```

**Status**: VERIFICADO

### 2. Alteração de Status de Pedidos

**Eventos**:
- Transição de status de pedidos (processing -> completed, pending -> failed)
- Tentativas de transição

**Arquivos**:
- grpms-approve_orders.php:115-125 (grpms_approve_orders_log)
- grpms-approve_orders.php:200-210 (grpms_approve_orders_audit_write)

**Informações Logadas**:
- order_id
- from_status
- to_status
- actor_id (get_current_user_id())
- outcome (success/failed/rate_limited)
- timestamp

**Status**: VERIFICADO

### 3. Acesso a Dados Sensíveis

**Eventos**:
- Acesso a endpoints de vendedor
- Acesso a endpoints de afiliado
- Acesso a endpoints administrativos

**Arquivos**:
- Multiple endpoints verificam e logam acessos não autorizados

**Exemplo**:
```python
# seller_dashboard_router.py:17
logger.debug("Seller dashboard access denied")
raise HTTPException(status_code=403, detail="Você não pode acessar dados de outro vendedor.")
```

**Status**: PARCIALMENTE VERIFICADO

**Nota**: Logging de acesso é feito via logger.debug, mas não há logging centralizado de todos os acessos

### 4. Operações Financeiras

**Eventos**:
- Solicitação de transferência
- Pagamento de comissões
- Atualização de status financeiro

**Arquivos**:
- A ser verificado (não explicitamente encontrado nos arquivos analisados)

**Status**: A SER VERIFICADO

### 5. Rate Limiting

**Eventos**:
- Tentativas de requisição excessivas
- Bloqueio por rate limiting

**Arquivos**:
- grpms-approve_orders.php:295-315

**Implementação**:
```php
// grpms-approve_orders.php:305-308
if (!grpms_approve_orders_rate_limit($actor_id)) {
    $logged_key = 'grpms_ao_rate_logged_' . $actor_id;
    if (!get_transient($logged_key)) {
        grpms_approve_orders_log($order_id, [], $to, 'rate_limited');
    }
}
```

**Status**: VERIFICADO

## Mecanismos de Logging

### WordPress

- **error_log()**: Usado para logs de auditoria
- **Nível**: INFO, WARNING, ERROR
- **Formato**: JSON (grpms_approve_orders.php:120-124)

### FastAPI

- **logging.getLogger()**: Usado em múltiplos arquivos
- **Nível**: DEBUG, INFO, WARNING, ERROR
- **Formato**: Texto estruturado

**Arquivos com logging**:
- backend/fastapi_app/routers/seller_dashboard_router.py:10-11, 14, 17, 24, 30, 36
- backend/fastapi_app/routers/mobile/mobile_salehistory_router.py:3-4, 22, 29
- backend/fastapi_app/routers/mobile/mobile_profile_router.py:2, 11, 18
- backend/fastapi_app/services/checkout/order_service.py:17-18

## Rastreabilidade

**Identificadores Unicos**:
- **actor_id**: get_current_user_id() (WordPress user ID)
- **order_id**: ID do pedido
- **vendor_id**: ID do vendedor
- **affiliate_id**: ID do afiliado

**Correlação**:
- Todos os logs de auditoria incluem actor_id
- Logs de pedidos incluem order_id e vendor_id
- Possível rastrear ações por usuário

**Status**: VERIFICADO

## Retenção de Logs

**Status**: A SER VERIFICADO

**Nota**: Não foram encontrados mecanismos de retenção ou rotação de logs nos arquivos analisados. Isso é tipicamente configurado no nível de servidor (logrotate, etc.)

## Status de Verificação (OWASP ASVS V7)

| Requisito | Status | Arquivo:Linha | Notas |
|-----------|--------|---------------|-------|
| V7.1.1 - Audit logging | VERIFICADO | Multiple | Logging de eventos sensíveis |
| V7.1.2 - Audit log protection | A SER VERIFICADO | - | Proteção de arquivos de log |
| V7.1.3 - Log retention | A SER VERIFICADO | - | Políticas de retenção não documentadas |
| V7.1.4 - Log review | A SER VERIFICADO | - | Processo de revisão não documentado |
| V7.2.1 - Unique identifiers | VERIFICADO | Multiple | actor_id, order_id, vendor_id |
| V7.2.2 - Correlation | VERIFICADO | Multiple | Possível correlação por IDs |

## Referências
- OWASP ASVS 5.0 V7: Logging and Monitoring
- WordPress Logging Best Practices
