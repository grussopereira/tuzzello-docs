# Arquitetura de Segurança - Tuzzello Marketplace

## Visão Geral

Este documento descreve a arquitetura de segurança do sistema Tuzzello Marketplace, incluindo limites de confiança, controles de segurança e mecanismos de proteção implementados para proteger dados de vendedores, afiliados e transações financeiras.

## Limites de Confiança (Trust Boundaries)

### 1. Frontend WordPress
- **Limite**: Interface web acessível por usuários finais
- **Confiança**: Nível médio - valida entrada de usuário, mas depende de backends para autorização
- **Controles**: 
  - Autenticação via WordPress (cookies de sessão)
  - Validação de nonce para requisições AJAX
  - Filtragem de entrada (sanitize_text_field, absint, etc.)
  - GRPMS_Access_Validator para controle de acesso baseados em funções

### 2. Backend FastAPI
- **Limite**: API interna acessível apenas do frontend WordPress
- **Confiança**: Alto - acessível apenas com segredo compartilhado
- **Controles**:
  - require_wordpress (X-GRPMS-WP-Secret header)
  - JWT RS256 para autenticação de usuário
  - Dependencies de autorização (get_current_user, require_mobile_user_id)

### 3. Banco de Dados
- **Limite**: Armazenamento de dados sensíveis
- **Confiança**: Alto - acesso controlado por aplicações
- **Controles**:
  - Conexões com credenciais segura
  - Consultas parametrizadas (prepared statements)
  - Isolamento por vendor_id/affiliate_id em consultas

### 4. Integração WooCommerce
- **Limite**: Integração com sistema de e-commerce
- **Confiança**: Médio - sistema de terceiros
- **Controles**:
  - Webhooks com verificação HMAC
  - Sincronização de pedidos com validação de integridade
  - Segredo compartilhado para chamadas internas

## Controles de Segurança Implementados

### Autenticação (OWASP ASVS V2)
- **V2.1**: Autenticação multifator (2FA/TOTP) via GRPMS_TOTP_Service
- **V2.2**: Gerenciamento de sessão com tokens JWT RS256
- **V2.3**: Validação de credenciais WordPress

### Autorização (OWASP ASVS V4)
- **V4.1**: RBAC implementado via WordPress roles (wcfm_vendor, yith_affiliate, shop_manager, administrator)
- **V4.2**: Controle de acesso fine-grained via GRPMS_Access_Validator
- **V4.3**: Isolamento de vendedor (vendor_id) em todos os endpoints de vendedor
- **V4.4**: Isolamento de afiliado (affiliate_id) em todos os endpoints de afiliado

### Proteção de Dados (OWASP ASVS V6)
- **V6.1**: Criptografia de dados em trânsito (HTTPS)
- **V6.2**: Não armazenamento de senhas em texto claro
- **V6.3**: Tokenização de dados sensíveis

### Validação de Entrada (OWASP ASVS V5)
- **V5.1**: Validação e sanitização de todos os parâmetros de entrada
- **V5.2**: Uso de prepared statements para consultas SQL
- **V5.3**: Validação de tipos e formatos

### Logging e Monitoramento (OWASP ASVS V7)
- **V7.1**: Auditoria de eventos sensíveis
- **V7.2**: Logging de tentativas de acesso não autorizado
- **V7.3**: Monitoramento de taxas de requisição

### Proteção de API (OWASP ASVS V8)
- **V8.1**: Rate limiting em endpoints sensíveis
- **V8.2**: Validação de conteúdo (Content-Type, size limits)
- **V8.3**: Proteção CSRF em formulários WordPress

## Mecanismos de Isolamento de Vendedores

### Princípio
Um vendedor NÃO pode ler ou alterar pedidos, comissões ou dados financeiros de outro vendedor, mesmo alterando IDs na API.

### Implementação
- **FastAPI**: Cada endpoint verifica se o user_id do token JWT corresponde ao vendor_id/affiliate_id
- **WordPress REST**: Filtragem por author = get_current_user_id() ou vendor_id = get_current_user_id()
- **AJAX Handlers**: Validação deroles e verificação de pertinência de dados

### Status de Verificação

Ver: SECURITY-ARCHITECTURE.md:35 (V4.3), SECURITY-ARCHITECTURE.md:36 (V4.4)

## Referências
- OWASP ASVS 5.0
- WordPress Security Best Practices
- FastAPI Security Documentation
