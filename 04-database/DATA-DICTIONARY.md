# Dicionário de dados
Propósito: descrever campos e regras observáveis nas definições `grpms_*` e nas tabelas WordPress/WooCommerce relacionadas; data: 2026-11-03; este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Convenções e limitações

Tipos abaixo são tipos SQL das definições em arquivo. “ID lógico” significa que o código guarda um identificador sem FK declarada. SQL de `backend: sql/auto_complete/` é explicitamente LAB ONLY/não executado; não comprova schema instalado. Para tabelas cujas definições não existem nos arquivos encontrados, os campos são **UNVERIFIED** em vez de inferidos. Não há definições completas de tabelas core WP/WooCommerce nos repositórios; seus campos abaixo só são descritos quando o código os referencia.

## Tabelas com definição observável

### Migrações do backend

| Tabela | Campos e significado observável | Chaves/regras | Evidência |
|---|---|---|---|
| `grpms_order_status_history` | `ID` BIGINT UNSIGNED auto incremento; `order_id`, `product_id` BIGINT UNSIGNED; `old_status`, `new_status`, `source` VARCHAR(32); `changed_at` DATETIME(6). Histórico de transição. | PK `ID`; índice `(order_id, changed_at)`; sem FK declarada. DDL referência LAB. | `backend: sql/auto_complete/001_auto_complete_history.sql:7-17` |
| `grpms_wc_status_outbox` | `ID`; `order_id`; `target_status`; `idempotency_key`; `state`; `attempts`; `next_attempt_at`; `locked_at`; `last_error_code`; `created_at`. Registro append-only de mensagem para alterar status; estado/retentativa são mutáveis. | PK; duas UNIQUE; índices para lease, vencimento e pedido; CHECK para status alvo `completed`, estados, lock, idempotência, ID positivo e token de erro. | `backend: sql/auto_complete/002_outbox_and_windows.sql:12-37` |
| `grpms_order_payment_windows` | `order_id`; `payment_confirmed_at`, `refund_window_ends_at`, `payout_release_at` (DATETIME(6), podem ser NULL); migration 004 adiciona `transaction_id` VARCHAR(255) NULL. | PK order; índices de janelas; checks de ordem temporal e piso de 15 dias para payout; trigger torna evidência de pagamento imutável e payout apenas extensível. | `backend: sql/auto_complete/002_outbox_and_windows.sql:43-76`; `004_webhook_deliveries.sql:15-24` |
| `grpms_payout_hold` | `ID`; `order_id`; `scope` VARCHAR(16); `reason` VARCHAR(64); `state` VARCHAR(16); `created_at`, `resolved_at`. Motivo de bloqueio de payout. | PK; índices por pedido/estado; checks de scope `order`, estado e consistência da resolução; resolução via UPDATE conforme comentário. | `backend: sql/auto_complete/002_outbox_and_windows.sql:78-97` |
| `grpms_wc_webhook_deliveries` | `delivery_id` VARCHAR(64) ASCII binário; `order_id`; `topic`; `received_at`; `outcome`; `payload_sha256` CHAR(64); `transaction_id` VARCHAR(255) nullable. | PK delivery; UNIQUE hash; checks de ID positivo e outcomes enumerados. | `backend: sql/auto_complete/004_webhook_deliveries.sql:3-14` |
| `grpms_alert_log` | `order_id` BIGINT UNSIGNED, `error_code` VARCHAR(64) ASCII binário, `last_sent_at` DATETIME(6). | UNIQUE `(order_id,error_code)`; linhas acumulam e pruning cabe ao migrator, conforme comentário. | `backend: sql/auto_complete/005_alert_log.sql:9-15` |
| `grpms_order_delivery` | `order_id` BIGINT UNSIGNED; `completed_at` DATETIME; `source` ENUM (`backend_status`, `wc_sync`, `repair_script`, `backfill`); `created_at` DATETIME. | `order_id` PK; comentários declaram append-only e sem FK. | `backend: sql/t2a/001_grpms_order_delivery.sql:2-11` |
| `refund_operation` | `id`, `order_id`, `wc_refund_id`, `requested_by`, `approved_by` IDs; `idempotency_key`, `reason_code`, `currency`; `kind`, `status`, `anchor_mode` enums; valores monetários em centavos; janelas e timestamps; `active_flag` gerado. | PK; UNIQUE pedido/refund, chave idempotente e pedido/flag ativo; índices pedido e status/data; checks de valores, reason e janela. Sem FK; LAB. | `backend: sql/t2a/002_refund_operation.sql:6-43` |
| `refund_line_allocation` | IDs de operação, pedido, linha principal e participante; `party`/`direction` enums; amount em centavos; currency; created_at. | PK; UNIQUE operação/linha/party/direction; sem FK, LAB. | `backend: sql/t2a/003_refund_line_allocation.sql:6-18` |
| `refund_fee_treatment` | operation_id; seq SMALLINT UNSIGNED; order_id; operator_fee_cents; currency; fee_state/debit_state enums; debit_cents; created_at. | PK; UNIQUE operação/seq; sem FK, LAB. | `backend: sql/t2a/004_refund_fee_treatment.sql:6-18` |
| `refund_audit` | operation_id nullable; order_id, actor_id; event enum; actor_role; before/after JSON; hashes IP/UA opcionais; created_at. | PK; índice pedido/data; append-only conforme comentário; sem FK, LAB. | `backend: sql/t2a/005_refund_audit.sql:6-19` |
| `grpms_order_approval_audit` | `id`, `actor_id`, `order_id` unsigned; `from_status`, `to_status`, `outcome`; `row_count`; `created_at`. | PK; índices ator/data, pedido e data; sem FK conforme comentário; migration requer aprovação. | `backend: sql/t07/001_grpms_order_approval_audit.sql:3-28` |

### Definições no frontend

| Tabelas | Campos e significado | Regras observáveis | Evidência |
|---|---|---|---|
| `grpms_training_questions`, `grpms_training_answers` | Pergunta ligada a curso/usuário e resposta ligada a pergunta/usuário; conteúdo textual, status e timestamps. | PK id; índices por curso, usuário, pergunta. | `frontend: includes/training/grpms-training.php:33-54` |
| `grpms_training_announcements` | curso, autor, título, conteúdo longo e timestamps. | PK id; índices por curso e autor. | `frontend: includes/training/grpms-training.php:55-66` |
| `grpms_training_enrollments` | usuário, curso, produto, pedido, vendedor; estado e período de acesso. | UNIQUE `(user_id,course_id,order_id)`; índices por status e IDs. | `frontend: includes/training/grpms-training.php:67-86` |
| `grpms_training_progress` | usuário/curso/módulo/lição; estado, percentual, posição em segundos, conclusão e timestamps. | UNIQUE `(user_id,lesson_id)`; índices usuário/curso e lição. | `frontend: includes/training/grpms-training.php:87-103` |
| `grpms_training_certificates` | UUID/hash/número, usuário/curso, nomes de estudante/curso/instrutor, totais de lições/duração e datas. | UNIQUE UUID e `(user_id,course_id)`; índices por curso/usuário. | `frontend: includes/training/grpms-training.php:104-124` |
| `grpms_kyc_submissions` | usuário, tipo/validade de documento, URLs de frente/verso/comprovante/selfie, estado, datas e usuário revisor, notas. | PK id; índices user, status, submitted_at. | `frontend: includes/admin/kyc/grpms-kyc.php:15-36` |
| `grpms_useract_audit` | `id`, `created_at`, `event`, `actor_id`, `target_id`. | PK id; índices target, actor e data. Prefixo `$wpdb->prefix`; criação condicional em `admin_init`. | `frontend: includes/registry/grpms-registry.php:807-840` |
| `grpms_order_locks` | `order_id`, `owner`, `expires_at`. | PK de pedido; índice expiração; DDL por `dbDelta`. | `frontend: includes/seller/grpms-seller.php:1192-1205` |

## Tabelas relacionadas WordPress/WooCommerce

| Tabela/abstração | Uso confirmado | Limites |
|---|---|---|
| `$wpdb->posts` / `wp_posts` | Lookup de produto/post; ex.: join por `p.ID` e instâncias de `$wpdb->posts`. | Não é prova de que pedido seja lido de `wp_posts`; prefixo literal `wp_posts` aparece em certos trechos. `frontend: includes/affiliation/grpms-affiliategeneral.php:188,356`; `tests/dashboard-scope-security.php:33` (teste somente). |
| `$wpdb->postmeta` / `wp_postmeta` | Manipulação de metadados de post e comentários de comissão; código também remove meta de lock legado. | Campos WP conhecidos não têm DDL neste escopo; chaves usadas variam por fluxo. `frontend: includes/seller/grpms-seller.php:1199-1201`; `includes/general/grpms-general.php:1051`. |
| `wp_wc_orders` (HPOS) | Atualização direta de `status` pela chave `id`, em caminho que também chama CRUD do WooCommerce. | Nome fixo `wp_`; não foram documentadas todas as colunas/relacionadas. `frontend: includes/general/grpms-general.php:1244-1257`. |
| WooCommerce CRUD de pedido | `wc_get_order(id)`, `get_status`, `update_status`, `save`; também `wc_get_orders` para busca por customer. | API abstrai data store; a configuração real e a estratégia de armazenamento não são verificáveis pelo código-fonte sozinho. `frontend: includes/seller/grpms-seller.php:1213-1219`; `includes/registry/grpms-registry.php:894-900`. |

## Gaps

UNVERIFIED: DDL completo de `grpms_main_orders`, payout/request/access tables e tabelas WP/WooCommerce; tipos e constraints das tabelas criadas fora das definições listadas; prefixo de produção; constraints FK físicas; colunas HPOS relacionadas e opção HPOS ativa. O backend contém referências a tabelas que não têm DDL correspondente no escopo (`grpms_main_orders`, `grpms_payouts`, `grpms_payout_items`); não inferir campos ausentes do código de consulta.
