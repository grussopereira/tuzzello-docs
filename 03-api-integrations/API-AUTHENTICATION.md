Propósito: explicar como a API emite e valida tokens e autentica integrações WordPress e vendedores.

Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nesta data.

# Autenticação

## JWT RS256

O backend assina JWT com RS256 e valida usando chave pública. A expiração é inserida como `exp`; a duração padrão é 15 minutos e pode ser configurada por `JWT_EXPIRE_MINUTES`. Não reproduza nem distribua as chaves. (`backend/fastapi_app/auth/jwt_rs256.py:5-30`.)

`POST /api/token/from-wp` aceita JSON `{"user_id": <inteiro positivo>}` e exige o segredo compartilhado WordPress. O backend lê capacidades/roles do WordPress, então retorna `access_token` com `user_id`, `roles` e `app`. Usuário sem roles válidas resulta em 403; falha inesperada resulta em 500. (`backend/fastapi_app/routers/token_router.py:10-42`.)

`POST /api/token` e `POST /api/token/login` foram aposentados e respondem 410 com orientação para autenticação de sessão WordPress. Não são caminhos de renovação. (`backend/fastapi_app/routers/token_router.py:13-16`.) Não há fluxo explícito de refresh token no código verificado; após expiração, a obtenção de novo token depende da ponte WordPress. Refresh dedicado: UNVERIFIED / não implementado no trecho verificado.

Rotas que usam `get_current_user` exigem bearer token; token inválido/expirado resulta 401 e `WWW-Authenticate: Bearer`. `get_current_user_optional` ignora ausência de token e retorna `None` quando falha a verificação. (`backend/fastapi_app/auth/dependencies.py:7-33`.)

## Segredo compartilhado WordPress

Rotas com `require_wordpress` esperam o header `X-GRPMS-WP-Secret`. O segredo do servidor vem de `GRPMS_WP_SHARED_SECRET`; configuração ausente ou com menos de 32 caracteres dá 503 e valor ausente/incorreto dá 401. A comparação usa `secrets.compare_digest`. Nunca coloque o valor do segredo na documentação. (`backend/fastapi_app/auth/wordpress_bridge.py:10-16`.)

## Identidade de vendedor assinada

As rotas que exigem identidade usam `X-GRPMS-Vendor-Id`, `X-GRPMS-Ts` e `X-GRPMS-Sig`. A chave configurada por `GRPMS_WP_IDENTITY_KEY` deve ter pelo menos 32 caracteres. O timestamp Unix deve estar dentro de 60 segundos. A assinatura é HMAC-SHA256 hexadecimal da mensagem `MÉTODO|caminho|query|vendor|timestamp`; a query é formada por `urlencode(sorted(request.query_params.multi_items()))`. Comparação é constante. Ausência/configuração inválida ou assinatura/timestamp/ID inválido resulta 503 ou 401 conforme o caso. (`backend/fastapi_app/auth/wordpress_bridge.py:18-35`.)

Exemplo estrutural, sem credenciais:

```http
X-GRPMS-Vendor-Id: <id decimal positivo>
X-GRPMS-Ts: <timestamp Unix atual>
X-GRPMS-Sig: <HMAC-SHA256 hexadecimal calculado no servidor cliente>
```

Não há endpoint de emissão ou renovação desta assinatura no backend verificado. A implementação correspondente no cliente deve ser consultada no código WordPress antes de reproduzir o cálculo; validade e fluxo de renovação além da janela de 60 segundos: UNVERIFIED.
