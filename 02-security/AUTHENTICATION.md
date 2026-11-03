# Autenticação - Tuzzello Marketplace

## Visão Geral

Este documento descreve os mecanismos de autenticação implementados no Tuzzello Marketplace, em conformidade com os requisitos do OWASP ASVS 5.0, Capítulo V2.

## Mecanismos de Autenticação

### 1. Autenticação WordPress (Frontend)

**Arquivo**: frontend/includes/api/grpms-rest-api.php

- **Mecanismo**: Sessão WordPress baseada em cookies
- **Fluxo**: 
  1. Usuário faz login via WordPress (wp_login)
  2. Sessão é estabelecida com cookie de autenticação
  3. WordPress gerencia estado de login via `is_user_logged_in()`
  4. ID do usuário é obtido via `get_current_user_id()`

- **Validação**: 
  - `grpms_rest_current_user_or_error()` (linha 26-34): Verifica se usuário está logado
  - `grpms_rest_user_has_role()` (linha 36-38): Verifica se usuário possui função específica

### 2. Autenticação JWT RS256 (Backend FastAPI)

**Arquivos**: 
- backend/fastapi_app/auth/jwt_rs256.py
- backend/fastapi_app/auth/dependencies.py

- **Algoritmo**: RS256 (Assinatura digital com chave privada)
- **Fluxo**:
  1. Frontend WordPress solicita token via `/api/token/from-wp` (token_router.py:27)
  2. Backend valida usuário WordPress e gera token JWT com claims:
     - `user_id`: ID do usuário WordPress
     - `roles`: Funções do usuário (wcfm_vendor, yith_affiliate, etc.)
     - `app`: Identificador da aplicação ('grpms_plugin')
  3. Token é usado em requisições para endpoints FastAPI via header `Authorization: Bearer <token>`

- **Validação**:
  - `get_current_user()` (dependencies.py): Valida token JWT e retorna claims
  - `get_current_user_optional()`: Versão opcional que não falha se não houver token

### 3. Autenticação de Integração WordPress-FastAPI

**Arquivo**: backend/fastapi_app/auth/wordpress_bridge.py

- **Mecanismo**: Segredo compartilhado (Shared Secret)
- **Header**: `X-GRPMS-WP-Secret`
- **Validação**: 
  - `require_wordpress()` (linha 7-10): Verifica se header corresponde ao segredo configurado
  - Segredo deve ter no mínimo 32 caracteres (linha 8)

## 2FA/TOTP

**Arquivo**: frontend/includes/api/grpms-backend-client.php

- **Implantação**: GRPMS_TOTP_Service
- **Uso**: 
  - Requerido para alteração de senha (`grpms_rest_profile_password`, linha 140-162)
  - Verificação via `GRPMS_TOTP_Service::verify_step_up()` (linha 157)
  - Código TOTP exigido se `GRPMS_TOTP_Service::is_required_for_user()` (linha 152)

## Expiração e Renovação

- **Tokens JWT**: Não implementado explicitamente (a ser verificado)
- **Sessões WordPress**: Gerenciadas pelo WordPress core
- **Tokens de API**: Gerados sob demanda, sem expiração explícita no código

## Status de Verificação (OWASP ASVS V2)

| Requisito | Status | Arquivo:Linha | Notas |
|-----------|--------|---------------|-------|
| V2.1.1 - Mecanismo de autenticação | VERIFICADO | Multiple | WordPress + JWT RS256 |
| V2.1.2 - Autenticação multifator | VERIFICADO | grpms-backend-client.php:152-157 | 2FA/TOTP para operções sensíveis |
| V2.1.3 - Armazenamento seguro de credenciais | VERIFICADO | WordPress core | Não armazenamos credenciais |
| V2.1.4 - Proteção contra brute force | PARCIAL | WordPress core | Depende de configuração WordPress |
| V2.1.5 - Mensagens de erro genéricas | VERIFICADO | Multiple | Sem exposição de informações |

## Referências
- OWASP ASVS 5.0 V2: Authentication
- WordPress Authentication Handbook
- JWT RFC 7519
