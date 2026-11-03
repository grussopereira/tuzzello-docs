Propósito: descrever a estratégia de QA observável nos testes versionados.
Data: 2026-11-03.
Escopo: descreve o HEAD de FRONTEND e BACKEND nesta data (c1bd75d e e53fb27).

# Estratégia de testes

## Inventário verificado

O inventário considera caminhos rastreados no Git em `tests/**` e `scripts/test-*`, sem executar testes: FRONTEND contém 55 arquivos e BACKEND 18. O FRONTEND inclui testes PHP e JavaScript para segurança, fluxos de dashboard, catálogo, perfil, treinamento e produto; por exemplo, `tests/access-control-security.php:1-12`, `tests/dashboard-ui.cjs:1-8` e `tests/product-workspace-browser.js:1-8`. O BACKEND inclui testes Python de serviços, checkout, webhook, reembolso, comissões e segurança; exemplos: `tests/test_wc_webhook.py:1-19`, `tests/test_t1a_commissions.py:1-12` e `tests/test_t2a_refunds.py:1-15`.

## Abordagens presentes

- Testes isolados com fixtures, mocks e bancos simulados. O teste de webhook declara fixtures sintéticas e conexão simulada (`BACKEND/tests/test_wc_webhook.py:1-39`).
- Asserções de contratos de segurança e de estrutura do código PHP/JavaScript ocorrem em arquivos do FRONTEND, como a verificação de nonce e capabilities em `tests/financial-ajax-security.php:23-31, 43-64`.
- Testes de propriedades e casos-limite da matemática de comissão estão em `BACKEND/tests/test_t1a_commissions.py:13-63`.
- Regressões de replay, concorrência e falhas transacionais de webhook estão em `BACKEND/tests/test_wc_webhook.py:150-191, 274-322`.
- Uma prova MariaDB está opt-in e exige banco descartável explicitamente configurado; não é execução padrão (`BACKEND/tests/test_wc_outbox_db.py:1-20`).

## Limites de evidência

O inventário informa arquivos versionados, não quantidade de casos executáveis nem resultado atual. Nenhuma suíte foi executada nesta tarefa, conforme solicitado. Configuração completa de CI, comandos oficiais, cobertura percentual e integração end-to-end de checkout são UNVERIFIED. As fixtures e asserts citados demonstram escopo pretendido dos testes, não sucesso de execução.
