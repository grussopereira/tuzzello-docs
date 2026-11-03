Propósito: registrar a cobertura de integração verificável entre componentes.
Data: 2026-11-03.
Escopo: descreve o HEAD de FRONTEND e BACKEND nesta data (c1bd75d e e53fb27).

# Testes de integração

## Webhook WooCommerce e outbox

O conjunto `BACKEND/tests/test_wc_webhook.py` exercita verificação e processamento com objetos simulados, banco falso transacional e payloads sintéticos (`:1-39`). Inclui replay de entrega e payload repetido (`:150-191`), concorrência de entregas iguais (`:274-290`), falha de gravação na outbox com rollback e resposta de erro (`:314-322`) e tentativas de retry (`:322-330`). A implementação inicia transação, registra entrega e retorna `duplicate` quando já finalizada (`BACKEND/fastapi_app/services/checkout/wc_webhook.py:134-155`); grava a outbox no caminho de conclusão (`:181-199`). Isso não comprova conexão com WooCommerce real.

O worker usa `httpx.MockTransport` segundo o cabeçalho do teste (`BACKEND/tests/test_wc_outbox_worker.py:1-22`); testa falhas e retries (`:243-251, 341-369`). A integração real com MariaDB é opt-in e restrita a servidor descartável local (`BACKEND/tests/test_wc_outbox_db.py:1-27`), incluindo concorrência de claim (`:47-82`). Não executar nesta tarefa.

## Importação e comissão

`BACKEND/tests/test_b_t6_dedup.py:52-85` verifica duas linhas do mesmo produto, retry sem nova inserção e colisão de índice com rollback. `BACKEND/tests/test_t1a_commissions.py:65-85` cobre integração da matemática com atribuição e sinalização de ativação. Os testes citados são offline; integração ponta a ponta entre checkout frontend, gateway, webhook e criação de pedido é UNVERIFIED.

## Reembolso

`BACKEND/tests/test_t2a_refunds.py:1-15, 60-83` exercita guarda e persistência usando SQLite em memória, não MariaDB DDL/grants. A rejeição parcial e gravação de auditoria são verificadas em `:108-115`; a implementação declara que não há writer de reembolso integral e emite apenas `partial_not_allowed` (`BACKEND/fastapi_app/services/checkout/refund_store.py:85-89, 123-138`). Rollback na falha de auditoria é verificado em `:266-286`. Integração com gateway para emitir estorno é UNVERIFIED.

## Fronteira FRONTEND/BACKEND

Os repositórios têm arquivos denominados `cloud-integration` (`FRONTEND/tests/cloud-integration.php` e `BACKEND/tests/test_cloud_integration.py`), mas não foi validado aqui um fluxo conectado entre instâncias. Conteúdo operacional e dependências reais são UNVERIFIED. Contagem do inventário: 55 arquivos rastreados FRONTEND e 18 BACKEND em `tests/**` e `scripts/test-*`.
