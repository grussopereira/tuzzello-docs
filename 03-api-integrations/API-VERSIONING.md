Propósito: registrar a política de versionamento e compatibilidade demonstrada no código e no histórico disponível.

Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend e backend nesta data.

# Versionamento

## Evidência atual

Não há prefixo de versão global nos caminhos montados em `main.py`; as rotas incluem `/api/...` sem marcador `/v1`. (`backend/fastapi_app/main.py:22-45`.) Os endpoints antigos `POST /api/token` e `/api/token/login` foram descontinuados operacionalmente no código e retornam 410, com orientação para autenticação de sessão WordPress. (`backend/fastapi_app/routers/token_router.py:13-16`.) Isso evidencia uma retirada pontual, não uma política geral de ciclo de vida.

O campo `version` da aplicação FastAPI não é definido ao criar `FastAPI()` (`backend/fastapi_app/main.py:22`); uma versão pública de API: UNVERIFIED. Os documentos e registros consultados não definem janela de compatibilidade, aviso prévio, suporte a versões paralelas ou cronograma de depreciação.

## Política

Política formal de versionamento, compatibilidade e depreciação: **TO BE DEFINED**. Não assumir compatibilidade semântica entre alterações. Antes de remover ou alterar contratos, registrar decisão aprovada e atualizar consumidores frontend.

## Versão OpenAPI deste contrato

Este arquivo usa `openapi: 3.1.0`, alvo compatível com o comportamento padrão documentado para FastAPI. A aplicação instancia `FastAPI()` sem substituição da versão (`backend/fastapi_app/main.py:22`), mas o `/openapi.json` em runtime não foi obtido; portanto a versão efetivamente emitida permanece UNVERIFIED. O OpenAPI 3.2.1, publicado em 10/09/2026, é a versão mais recente indicada pela especificação oficial consultada: https://spec.openapis.org/oas/v3.2.1.html. Não se assume compatibilidade de ferramenta com 3.2.1. A dependência local é FastAPI 0.141.1 (`backend/requirements-cloud.txt:1`); o alvo 3.1.0 evita declarar suporte não verificado.
