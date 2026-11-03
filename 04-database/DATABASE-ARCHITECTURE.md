# Arquitetura do banco de dados
Propósito: descrever os grupos de tabelas e suas responsabilidades conforme o código; data: 2026-11-03; este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Escopo e fontes

Esta documentação foi derivada dos arquivos do repositório, sem conexão a banco de dados. As referências `frontend:` e `backend:` apontam para caminhos relativos às raízes `/home/gilberto/Projects/Tuzzello-AI-Lab/frontend` e `/home/gilberto/Projects/Tuzzello-AI-Lab/backend`; os números indicam linhas no HEAD inspecionado. O prefixo WordPress pode ser configurável, mas alguns trechos usam `wp_` literalmente.

## Grupos de tabelas

| Grupo | Responsabilidade observável | Evidência |
|---|---|---|
| Pedidos, comissões e pagamentos do marketplace | `grpms_main_orders` mantém estado e valores de comissão associados ao pedido; tabelas de solicitação e auditoria de payout tratam solicitações e repasses. | backend: `sql/b_t6/002_swap_unique_order_product_to_order_item.sql:39-43`; frontend: `includes/financial/grpms-payment.php:1471-1478, 1632-1659` |
| Entrega e reembolsos | `grpms_order_delivery` associa entrega ao pedido; `refund_operation`, `refund_line_allocation`, `refund_fee_treatment` e `refund_audit` registram operação e distribuição de reembolso. | backend: `sql/t2a/001_grpms_order_delivery.sql:1-`; `sql/t2a/002_refund_operation.sql:1-`; `sql/t2a/003_refund_line_allocation.sql:1-`; `sql/t2a/004_refund_fee_treatment.sql:1-`; `sql/t2a/005_refund_audit.sql:1-` |
| Integração de status e pagamento | Histórico, outbox, janelas de pagamento, bloqueios de payout e recebimentos de webhook dão suporte a fluxos de status e evidências de pagamento. As migrações `auto_complete` dizem LAB ONLY/não executadas. | backend: `sql/auto_complete/001_auto_complete_history.sql:1-17`; `002_outbox_and_windows.sql:1-99`; `004_webhook_deliveries.sql:1-24` |
| Auditoria de aprovação e transações | `grpms_order_approval_audit` registra evento de aprovação; migração separada adiciona unicidade de transação em payout. | backend: `sql/t07/001_grpms_order_approval_audit.sql:15-28`; `sql/t07/003_unique_operator_transaction_id.sql:1-` |
| Acesso e identidade | `grpms_access_restrict` e `grpms_access_audit` são usados pelo normalizador de permissões; auditoria de ações de usuário registra ator/alvo. | frontend: `includes/admin/access/classes/class-grpms-access-normalizer.php:39,179-214`; `includes/registry/grpms-registry.php:807-840` |
| Treinamento | Perguntas, respostas, anúncios, inscrições, progresso e certificados relacionam usuários, cursos, módulos/lições, produtos e pedidos por IDs. | frontend: `includes/training/grpms-training.php:33-124` |
| KYC | `grpms_kyc_submissions` mantém submissões, estado de revisão, referências de arquivos e notas. | frontend: `includes/admin/kyc/grpms-kyc.php:5-36` |
| Locks operacionais | `grpms_order_locks` implementa lock temporário indexado pelo ID de pedido. | frontend: `includes/seller/grpms-seller.php:1192-1205` |

## Relação com WordPress e WooCommerce

O identificador de pedido é usado em tabelas GRPMS como chave lógica (por exemplo, `order_id` em janelas/outbox, `order_id` na inscrição de treinamento e `main_order_id` em itens de payout). O código mostrado não declara chaves estrangeiras para as tabelas centrais de pedido; integridade referencial depende de IDs e regras da aplicação. Veja [ER-DIAGRAM.md](ER-DIAGRAM.md).

O frontend usa `wc_get_order()` e objetos de pedido em múltiplos fluxos (`frontend: includes/seller/grpms-seller.php:1210-1218`; `includes/checkout/grpms-checkout.php:1801-`), que é uma interface CRUD do WooCommerce compatível com seus data stores. Porém, há também uma atualização SQL direta à tabela fixa `wp_wc_orders` (`frontend: includes/general/grpms-general.php:1244-1249`) e leituras/escritas legadas em `wp_posts`/`wp_postmeta` para outros casos. A documentação do repositório, portanto, não permite afirmar compatibilidade completa com HPOS: há caminho HPOS direto, caminho CRUD, e compatibilidade/consistência do conjunto não confirmadas. Não se encontrou prova de leitura direta de pedido a partir de `wp_posts`/`wp_postmeta` nesse fluxo de sincronização; referências dessas tabelas não são, por si, armazenamento do pedido.

## Limites

As migrações não comprovam quais scripts foram aplicados nem o estado real do banco. Migrações `auto_complete` explicitamente não executadas são propostas laboratoriais, não schema implantado. O backend também contém tabelas `grpms_*` referenciadas em código ou SQL cuja definição não está no conjunto examinado; veja [DATA-DICTIONARY.md](DATA-DICTIONARY.md). UNVERIFIED: engines, collation, prefixo efetivo, versões de WordPress/WooCommerce, modo HPOS ativo e aplicação das migrações.
