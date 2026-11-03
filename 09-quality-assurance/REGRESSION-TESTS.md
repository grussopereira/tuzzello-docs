Propósito: mapear regressões cobertas pelos testes existentes.
Data: 2026-11-03.
Escopo: descreve o HEAD de FRONTEND e BACKEND nesta data (c1bd75d e e53fb27).

# Testes de regressão

Contagem baseada em arquivos rastreados: 55 no FRONTEND e 18 no BACKEND, sob `tests/**` e `scripts/test-*`; contagem de arquivos não equivale a casos nem resultado de execução.

Regressões evidenciadas no BACKEND incluem duplicação e retry de importação (`tests/test_b_t6_dedup.py:52-85`), invariantes da matemática de comissão (`tests/test_t1a_commissions.py:13-63`), repetição/concorrência de webhook (`tests/test_wc_webhook.py:150-191, 274-290`) e rollback ao falhar gravação associada (`:314-322`). Implementação de deduplicação por entrega/payload está em `fastapi_app/services/checkout/wc_webhook.py:136-155`. Regras de reembolso são exercitadas por casos dourados e rejeição parcial (`tests/test_t2a_refunds.py:84-115`); a implementação registra a rejeição parcial e a auditoria (`fastapi_app/services/checkout/refund_store.py:121-138`).

No FRONTEND, há regressões focadas em segurança de payout, transição de status e proteção de dados; por exemplo, `tests/payout-duplicate-guard.php:144-148` e `tests/commission-status-downgrade-security.php:23-53`.

O briefing financeiro exige cinco cenários: (1) pagamento duplicado tem cobertura parcial de webhook/payout, mas teste de cobrança externa é AUSENTE (`BACKEND/tests/test_wc_webhook.py:150-191`; `FRONTEND/tests/payout-duplicate-guard.php:144-148`); (2) webhook repetido EXISTE (`BACKEND/tests/test_wc_webhook.py:150-191, 274-290`); (3) múltiplos cliques de checkout AUSENTE no inventário verificado; (4) reembolso parcial EXISTE como rejeição (`BACKEND/tests/test_t2a_refunds.py:108-115`); (5) falha durante distribuição de comissão AUSENTE no escopo identificado, pois rollback de webhook não equivale a distribuição (`BACKEND/tests/test_wc_webhook.py:314-322`).

Suítes não executadas nesta tarefa; resultado do HEAD e estado de CI são UNVERIFIED.
