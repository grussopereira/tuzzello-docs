Propósito: definir critérios verificáveis de aceite para QA documental e lacunas financeiras.
Data: 2026-11-03.
Escopo: descreve o HEAD de FRONTEND e BACKEND nesta data (c1bd75d e e53fb27).

# Critérios de aceite

## Evidência do código

- Cada afirmação sobre implementação deve apontar arquivo e linhas do HEAD; exemplos de matemática são `BACKEND/tests/test_t1a_commissions.py:13-63` e exemplos de webhook são `BACKEND/tests/test_wc_webhook.py:150-191`.
- O inventário declara seu método e tamanho: 55 arquivos rastreados no FRONTEND e 18 no BACKEND sob `tests/**` e `scripts/test-*`.
- Testes não foram executados nesta tarefa. Resultado de execução e CI devem permanecer UNVERIFIED.
- Não se deve transformar assert de fixture em garantia de produção. O teste webhook usa doubles (`BACKEND/tests/test_wc_webhook.py:1-39`); probe MariaDB exige opt-in (`BACKEND/tests/test_wc_outbox_db.py:19-27`).

## Aceite da cobertura financeira exigida

| Critério | Estado verificado |
|---|---|
| Distinguir repetição de webhook de nova cobrança | Coberto para replay/dedup de webhook em `BACKEND/tests/test_wc_webhook.py:150-191, 274-290` e implementação `BACKEND/fastapi_app/services/checkout/wc_webhook.py:136-155`; cobrança do provedor UNVERIFIED. |
| Validar cliques repetidos no checkout sem duplicar pedido/pagamento | AUSENTE no inventário de testes examinado; teste de UI e proteção efetiva UNVERIFIED. |
| Especificar tratamento de reembolso parcial | Teste existente de rejeição parcial/auditoria em `BACKEND/tests/test_t2a_refunds.py:108-115`; implementação em `BACKEND/fastapi_app/services/checkout/refund_store.py:121-138`; execução completa de estorno UNVERIFIED. |
| Provar atomicidade quando distribuição de comissão falha em etapa intermediária | AUSENTE no escopo examinado. A falha na outbox (`BACKEND/tests/test_wc_webhook.py:314-322`) não satisfaz este critério. |
| Identificar pedido repetido na solicitação de payout | Teste de lista duplicada deduplicada em `FRONTEND/tests/payout-duplicate-guard.php:144-148`; implementação usa `array_unique` em `FRONTEND/includes/financial/grpms-payment.php:814-821`; isso não equivale a transação de cobrança externa. |

## Itens UNVERIFIED

Comandos oficiais de execução, pipeline de CI, resultados de execução, cobertura percentual, integrações com provedores, fluxo frontend-backend real e requisitos de performance não foram estabelecidos por evidência citada neste conjunto.
