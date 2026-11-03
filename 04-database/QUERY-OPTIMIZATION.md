# Otimização de consultas
Propósito: inventariar índices declarados e consultas críticas visíveis no código, sem executar EXPLAIN ou consultar banco; data: 2026-11-03; este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Índices definidos em arquivos

| Grupo | Índices relevantes | Padrão de consulta suportado/evidência |
|---|---|---|
| Outbox (LAB) | UNIQUE idempotency; UNIQUE order/status; `(state,locked_at)`; `(state,next_attempt_at,ID)`; `(order_id,created_at)`. | Deduplicação, lease, seleção de pendentes vencidos e histórico por pedido; `backend: sql/auto_complete/002_outbox_and_windows.sql:23-28`. |
| Janelas e holds (LAB) | `(refund_window_ends_at,order_id)`, `(payout_release_at,order_id)`, `(order_id,scope,state)`, `(state,created_at,ID)`. | Varredura de prazos e gate por pedido; `backend: sql/auto_complete/002_outbox_and_windows.sql:48-50,88-90`. |
| Webhook (LAB) | PK `delivery_id`; UNIQUE `payload_sha256`. | Idempotência por entrega/payload; `backend: sql/auto_complete/004_webhook_deliveries.sql:4-13`. |
| Refund | UNIQUE de operação e alocação/taxa; `(order_id)`, `(status,created_at)`, `(order_id,created_at)`. | Leituras por pedido/status e auditoria temporal; `backend: sql/t2a/002_refund_operation.sql:28-32`; `003_refund_line_allocation.sql:17`; `004_refund_fee_treatment.sql:17`; `005_refund_audit.sql:18`. |
| Treinamento | Índices por curso/usuário, question_id, user/status, course/status, product/order/vendor, user/course, lesson e certificate course/user. | Consultas por aluno, curso e lição; `frontend: includes/training/grpms-training.php:41-43,51-53,63-65,80-85,99-123`. |
| KYC/auditoria/locks | KYC por user/status/submitted; user audit por target/actor/data; order lock por expiry. | Revisão e limpeza/listagens associadas; `frontend: includes/admin/kyc/grpms-kyc.php:29-32`; `includes/registry/grpms-registry.php:823-826`; `includes/seller/grpms-seller.php:1196-1198`. |
| Approval audit e payout transação | Índices de order/actor/time; UNIQUE operator transaction ID. | Consulta de auditoria e deduplicação transacional; `backend: sql/t07/001_grpms_order_approval_audit.sql:24-28`; `003_unique_operator_transaction_id.sql:10-`. |

## Consultas críticas encontradas

- Payout procura pedido por `ID`, valida status/refund/entrega e grava paid date sob lock advisory com UPDATE condicional (`frontend: includes/financial/grpms-payment.php:1471-1478,1861-1868`). A definição/índices de `grpms_main_orders` não estão no conjunto documentado; adequação não pode ser julgada.
- Relatórios de payout fazem joins de `grpms_payouts`, `grpms_payout_items` e `grpms_main_orders` (`frontend: includes/financial/grpms-payment.php:2467-2479,2740-2742`). Índices das tabelas de payout não foram confirmados por DDL nesta inspeção.
- Outbox expõe índices diretamente alinhados com lease e vencimento, porém a migration é LAB ONLY; não implica que worker ou índice estejam ativos (`backend: sql/auto_complete/002_outbox_and_windows.sql:1-28`).
- Fluxos de transação administrativa fazem re-leituras `FOR UPDATE`; o objetivo é consistência sob concorrência, não necessariamente throughput (`frontend: includes/financial/grpms-admin-request-review.php:15-30`).

## Limites

Não foi executado EXPLAIN e nenhuma conexão foi aberta. Não se recomenda índice novo sem esquema efetivo, cardinalidade e plano observados em ambiente autorizado. UNVERIFIED: índices atualmente instalados, seletividade, volume de dados, planos, estatísticas, lentidão medida e versão/engine do banco. SQL de migração não comprova implantação.
