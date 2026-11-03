# Integridade dos dados
Propósito: registrar chaves, regras de consistência, transações e locks comprovados no código; data: 2026-11-03; este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Chaves e unicidade

As migrations implementam PKs e UNIQUE constraints em áreas delimitadas: idempotência de outbox (`idempotency_key` e `(order_id,target_status)`), unicidade de operação de reembolso por pedido/refund, token idempotente e flag ativo; alocação única por operação/linha/parte/direção; seq de fee por operação; identidade de inscrição e progresso de treinamento; UUID e curso/usuário do certificado. Fontes: `backend: sql/auto_complete/002_outbox_and_windows.sql:23-27`; `sql/t2a/002_refund_operation.sql:28-32`; `003_refund_line_allocation.sql:17`; `004_refund_fee_treatment.sql:17`; `frontend: includes/training/grpms-training.php:80-121`.

Não há FK física declarada nas migrations citadas para tabelas centrais WordPress/WooCommerce ou entre várias tabelas GRPMS. Relações são conferidas por joins/IDs e regras de aplicação; existência do índice ou de um ID no código não prova integridade referencial.

## Checks e regras

Outbox LAB valida destino `completed`, conjunto de estados, associação de lock para `processing`, formato idempotente, ID positivo e código de erro em token ASCII maiúsculo (`backend: sql/auto_complete/002_outbox_and_windows.sql:29-36`). Janelas validam ordem temporal e payout com piso de 15 dias; trigger bloqueia alteração de evidência de pagamento já preenchida e redução/remoção de data de liberação (`:51-75`). Payout hold limita scope/estado e exige `resolved_at` coerente (`:91-96`). São definições de arquivo LAB, não confirmação de constraints instaladas.

## Transações e locks

Revisão de solicitação administrativa usa `START TRANSACTION`, carrega solicitação e afiliados com `FOR UPDATE`, faz `COMMIT`, e executa `ROLLBACK` na falha (`frontend: includes/financial/grpms-admin-request-review.php:10-64`). Normalizador de permissões combina lock advisory `GET_LOCK`, transação, leitura bloqueada, postcondição e rollback (`frontend: includes/admin/access/classes/class-grpms-access-normalizer.php:39,179-214`; `docs/import-access-policy.php:51-89`).

Payout usa locks advisory por solicitação e transação externa, com checagem de elegibilidade, unicidade de transação e escrita condicional em `grpms_main_orders`; trechos liberam locks em cleanup/shutdown (`frontend: includes/financial/grpms-payment.php:1715-1742,1814-1829,1861-1868`). O próprio código observa que lock advisory depende da conexão e reconexão pode removê-lo (`:842-843`). Esse fluxo de payout não demonstra atomicidade transacional abrangente entre todas as tabelas; possui compensação seletiva em caso de falha (`:1848-1859`).

## Consistência dos pedidos e HPOS

Fluxos CRUD via `wc_get_order()` delegam persistência ao WooCommerce. Um caminho sincroniza status com atualização direta em `wp_wc_orders` e também chama `update_status()`/`save()` (`frontend: includes/general/grpms-general.php:1221-1259`). O código não evidencia uma transação única cobrindo tabela GRPMS e armazenamento WooCommerce, nem confirma sincronização HPOS. UNVERIFIED: concorrência e consistência cross-store efetivas em produção.

## Riscos/gaps observáveis

- SQL com ID de pedido sem FK permite registros órfãos se operação não verificar pedido.
- Os arquivos `auto_complete` se identificam como LAB ONLY/não aplicados; constraints descritas podem não existir em produção.
- Algumas leituras/escritas usam prefixo literal `wp_`, enquanto outras obtêm prefixo de `$wpdb`; instalações com prefixo customizado podem divergir. Exemplo: `frontend: includes/general/grpms-general.php:1244-1248`.
- Engine das tabelas efetivas, isolamento e comportamento de locks são UNVERIFIED sem consultar ambiente (proibido neste trabalho).
