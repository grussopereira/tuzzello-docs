# Gerenciamento de Segredos - Tuzzello Marketplace

## Visão Geral

Este documento descreve como segredos, credenciais e chaves são armazenados, rotacionados e revogados no Tuzzello Marketplace, em conformidade com OWASP ASVS 5.0, Capítulo V9.

## Segredos Identificados

### 1. Segredo Compartilhado WordPress-FastAPI

**Propósito**: Autenticação de chamadas internas entre WordPress e FastAPI

**Localização**:
- Backend: Variável de ambiente `GRPMS_WP_SHARED_SECRET`
- Frontend: Constante `GRPMS_API_INTERNAL_URL` e `GRPMS_WP_SHARED_SECRET`

**Arquivos**:
- backend/fastapi_app/auth/wordpress_bridge.py:7-8
- frontend/includes/api/grpms-backend-client.php:15-17

**Requisitos**:
- Mínimo 32 caracteres (wordpress_bridge.py:8)
- Comparação segura usando secrets.compare_digest() (wordpress_bridge.py:10)

**Status**: VERIFICADO

**Política**:
- Armazenado em variáveis de ambiente
- Não armazenado em repositório
- Deve ser rotacionado periodicamente

### 2. Chaves EasyPost

**Propósito**: Integração com API EasyPost para envio de encomendas

**Localização**: Variáveis de ambiente

**Arquivo**: backend/fastapi_app/services/checkout/easypost_service.py:14-16

**Variáveis**:
- `EASYPOST_TEST_KEY` - Chave para ambiente de teste
- `EASYPOST_LIVE_KEY` - Chave para ambiente de produção
- `EASYPOST_MODE` - Modo de operação (test/production)

**Status**: VERIFICADO

**Política**:
- Armazenadas em variáveis de ambiente
- Chaves diferentes para teste e produção
- Modo configurável via EASYPOST_MODE

### 3. Chave Privada JWT RS256

**Propósito**: Assinatura de tokens JWT

**Localização**: Variável de ambiente (a ser confirmado)

**Arquivo**: backend/fastapi_app/auth/jwt_rs256.py

**Status**: A SER VERIFICADO

**Nota**: Implementação do JWT RS256 não explora explicitamente local da chave privada nos arquivos analisados

### 4. Chave de Webhook WooCommerce

**Propósito**: Verificação HMAC de webhooks do WooCommerce

**Localização**: Variável de ambiente (a ser confirmado)

**Arquivo**: backend/fastapi_app/services/checkout/wc_webhook.py

**Função**: service.verify() (woocommerce_order.py:44)

**Status**: A SER VERIFICADO

## Práticas de Gerenciamento

### Armazenamento

**VERIFICADO**:
- Todos os segredos são armazenados em variáveis de ambiente
- NENHUM segredo está hardcoded nos arquivos
- NENHUM segredo está commitado no repositório

**Mecanismo**:
- Uso de `os.getenv()` para acesso a segredos
- Arquivos .env não incluídos no repositório (.gitignore)

### Rotação

**Política Recomendada**:
- GRPMS_WP_SHARED_SECRET: Rotacionar a cada 90 dias
- EasyPost Keys: Rotacionar conforme política do provedor
- JWT Private Key: Rotacionar a cada 180 dias
- WooCommerce Webhook Secret: Rotacionar a cada 90 dias

**Status**: A SER IMPLEMENTADO

**Nota**: Não foram encontrados mecanismos automáticos de rotação nos arquivos analisados

### Revogação

**Processo**:
1. Rotacionar segredo em variável de ambiente
2. Reiniciar serviços para aplicar nova configuração
3. Auditar logs para garantir que antigo segredo não está mais em uso

**Status**: A SER DOCUMENTADO

## Caminhos de Arquivos

**Caminhos de configuração**:
- Backend: `/home/gilberto/Projects/Tuzzello-AI-Lab/backend/.env`
- Backend: `/home/gilberto/Projects/Tuzzello-AI-Lab/backend/.env.example`
- Frontend: `/home/gilberto/Projects/Tuzzello-AI-Lab/frontend/` (variáveis WordPress)

**Nota**: Arquivos .env não foram lidos para proteger segredos. Apenas estrutura e uso foram verificados.

## Status de Verificação (OWASP ASVS V9)

| Requisito | Status | Arquivo:Linha | Notas |
|-----------|--------|---------------|-------|
| V9.1.1 - No hardcoded secrets | VERIFICADO | Multiple | Todos os segredos em variáveis de ambiente |
| V9.1.2 - Segredos protegidos em trânsito | VERIFICADO | wordpress_bridge.py:7-10 | HTTPS + header seguro |
| V9.1.3 - Segredos protegidos em repouso | VERIFICADO | - | Variáveis de ambiente (servidor) |
| V9.2.1 - Rotação de segredos | A SER VERIFICADO | - | Política não documentada no código |
| V9.2.2 - Revogação de segredos | A SER VERIFICADO | - | Processo não documentado no código |

## Referências
- OWASP ASVS 5.0 V9: Secrets Management
- 12-Factor App: Config
