Propósito: catalogar áreas de teste existentes e os cenários financeiros exigidos no briefing.
Data: 2026-11-03.
Escopo: descreve o HEAD de FRONTEND e BACKEND nesta data (c1bd75d e e53fb27).

# Casos de teste e cobertura existente

Inventário: 55 arquivos rastreados em `tests/**` e `scripts/test-*` no FRONTEND e 18 no BACKEND. Contagem reproduzível pela listagem Git, sem incluir dependências locais. Nenhuma suíte foi executada.

## Casos evidenciados

| Área | Evidência existente no código de teste |
|---|---|
| Controle de acesso | FRONTEND valida ações, capacidades e nonce (`tests/access-control-security.php:1-12`, `tests/financial-capability-floor-security.php:1-16`). |
| Revisão administrativa | Script rejeita decisão duplicada (`scripts/test-admin-request-review.php:48`). |
| Comissão | BACKEND testa casos dourados, limites, entradas inválidas e propriedade de soma (`tests/test_t1a_commissions.py:13-63`). |
| Reimportação/deduplicação de itens | BACKEND testa retry sem nova inserção, conflito e payload duplicado (`tests/test_b_t6_dedup.py:61-85`). |
| Webhook | BACKEND testa repetição da mesma entrega, mesmo payload com ID novo, concorrência idêntica e falha de persistência com rollback (`tests/test_wc_webhook.py:150-191, 274-322`). |
| Reembolso parcial | BACKEND verifica que pedido parcial é rejeitado e auditado (`tests/test_t2a_refunds.py:108-115`). |
| Payouts | FRONTEND testa lista de pedidos repetidos deduplicada na solicitação (`tests/payout-duplicate-guard.php:144-148`); implementação também aplica `array_unique` e revalida elegibilidade (`includes/financial/grpms-payment.php:814-821, 909-916`). Isso não cobre cobrança de provedor. |

## Cenários financeiros obrigatórios

| Cenário | Estado | Evidência / lacuna |
|---|---|---|
| Pagamentos duplicados | Parcial | Repetição de webhook é tratada como duplicada em `BACKEND/tests/test_wc_webhook.py:150-191, 274-290`, conforme implementação que reconhece entrega/hash existente (`BACKEND/fastapi_app/services/checkout/wc_webhook.py:136-155`). Lista repetida de payout é deduplicada no FRONTEND (`tests/payout-duplicate-guard.php:144-148`; implementação em `includes/financial/grpms-payment.php:814-821`). Teste explícito de cobrança duplicada no provedor: AUSENTE no inventário verificado; comportamento externo UNVERIFIED. |
| Webhooks repetidos | Existe | `BACKEND/tests/test_wc_webhook.py:150-191` cobre entrega repetida e payload igual sob ID novo; linhas 274-290 cobrem concorrência. |
| Múltiplos cliques no checkout | AUSENTE | Não foi localizado caso de teste de clique repetido/submissão concorrente de checkout na lista rastreada do FRONTEND; a existência de proteção na interface é UNVERIFIED. |
| Reembolso parcial | Existe como rejeição | `BACKEND/tests/test_t2a_refunds.py:108-115` verifica rejeição de reembolso parcial; implementação registra estado rejeitado e auditoria (`BACKEND/fastapi_app/services/checkout/refund_store.py:121-138`). `BACKEND/tests/test_wc_webhook.py:175-183` reconhece impedimentos de reembolso. Fluxo de reembolso parcial concluído com distribuição proporcional: AUSENTE/UNVERIFIED. |
| Falha durante distribuição de comissão | AUSENTE / cobertura específica não localizada | `BACKEND/tests/test_t1a_commissions.py:65-85` testa integração da função de comissão; não evidencia falha no meio da distribuição financeira e recuperação/atomicidade. O teste de rollback de webhook em `BACKEND/tests/test_wc_webhook.py:314-322` é outra operação. |

O termo “existe” descreve somente o escopo explícito dos testes citados, não execução aprovada nem comportamento em produção.
