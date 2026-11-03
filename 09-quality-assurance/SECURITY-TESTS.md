Propósito: resumir controles de segurança cobertos pelos testes versionados.
Data: 2026-11-03.
Escopo: descreve o HEAD de FRONTEND e BACKEND nesta data (c1bd75d e e53fb27).

# Testes de segurança

O inventário contém 55 arquivos de teste rastreados no FRONTEND e 18 no BACKEND (padrões `tests/**` e `scripts/test-*`). O FRONTEND possui testes com foco explícito em controle de acesso, proteção financeira, sessão e dados de perfil; exemplos verificáveis: `tests/access-control-security.php:1-12`, `tests/financial-ajax-security.php:23-64`, `tests/dashboard-token-exposure-security.php:1-10` e `tests/profile-postal.php:1-10`.

Os testes financeiros verificam capability/nonce para ações AJAX (`FRONTEND/tests/financial-ajax-security.php:43-64, 97-101`) e controles de capability em payout (`FRONTEND/tests/payout-audit-capability-security.php:1-38`). A implementação confere nonce e capability antes de processar solicitação e remove IDs repetidos (`FRONTEND/includes/financial/grpms-payment.php:773-821`). Proteções contra downgrade de status e pagamento/refund elegível aparecem em `FRONTEND/tests/commission-status-downgrade-security.php:23-53`.

No BACKEND, `tests/test_wc_webhook.py:1-19` usa segredo artificial e payload de teste; casos de assinatura/replay e entrega conflitante aparecem em `:150-191, 694-714`. A implementação valida a entrega por chave/hash e detecta conflito (`fastapi_app/services/checkout/wc_webhook.py:136-155`). O teste do worker usa transporte HTTP simulado e valores canário de teste (`tests/test_wc_outbox_worker.py:1-22`). O teste de reembolso descreve explicitamente SQLite e fixtures offline, sem validar DDL/grants reais (`tests/test_t2a_refunds.py:1-15`).

Não foram lidos ou reproduzidos valores secretos ou pessoais. Testes dinâmicos não foram executados. Modelo de ameaça completo, cobertura de dependências, configuração de análise estática e conformidade normativa são UNVERIFIED; este documento não declara conformidade legal.
