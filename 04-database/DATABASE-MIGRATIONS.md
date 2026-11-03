# Migrações do banco de dados
Propósito: documentar como alterações de schema são controladas pelo código e pelos arquivos SQL; data: 2026-11-03; este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Localização e convenções

No backend, SQL versionado está sob `sql/`, organizado por área (`auto_complete`, `t2a`, `b_t6`, `t07`). Exemplos incluem `001_...sql` e pares com sufixo `.rollback.sql`; alguns diretórios têm README. O padrão numérico indica sequência do conjunto, mas não há um executor universal confirmado. Evidência: arquivos `backend: sql/auto_complete/001_auto_complete_history.sql`, `sql/t2a/README.md`, `sql/b_t6/002_swap_unique_order_product_to_order_item.sql`.

No frontend, algumas tabelas são criadas/atualizadas em runtime com `dbDelta()` e versão em option/transient: treinamento (`frontend: includes/training/grpms-training.php:20-31,125-`), auditoria de usuário (`includes/registry/grpms-registry.php:812-840`), KYC (`includes/admin/kyc/grpms-kyc.php:10-36`) e locks (`includes/seller/grpms-seller.php:1185-1205`). Isso não constitui histórico versionado único; a política global de versões/ordenação não está especificada.

## Aplicação e rollback

As migrations `sql/auto_complete` são explicitamente descritas como LAB ONLY, NOT EXECUTED ou exigindo aprovação/migração parametrizada. Grants e instruções SQL nelas são referências manuais, não execução automática pelo app (`backend: sql/auto_complete/003_grants.reference.sql:1-13`). Não aplicar ou considerar implantação concluída com base apenas na presença dos arquivos.

Os rollbacks auto_complete são gates manuais: exigem parar todos os escritores, confirmar banco/tabelas, verificar contagens vazias e nunca remover dados para satisfazer gate. Arquivo rollback pode ser no-op até operador executar manualmente DDL comentado (`backend: sql/auto_complete/002_outbox_and_windows.rollback.sql:1-19`; `004_webhook_deliveries.rollback.sql:1-21`). SQL DDL MariaDB implicitamente commita, conforme alertas desses arquivos.

Outras áreas trazem checagens e etapas de rollback/roll-forward específicas; por exemplo, `b_t6` troca índice único após checagem de duplicidades e tem script separado para restaurar índice anterior (`backend: sql/b_t6/001_check_duplicate_order_item.sql:1-`; `002_swap_unique_order_product_to_order_item.sql:1-`; `003_rollback_restore_uq_order_product.sql:1-`). A prática exata deve seguir o README e checklist de cada conjunto. Não há prova de runner que aplique uniformemente SQL de todas as pastas.

## Lacunas conhecidas

- Não existe inventário verificável de migrations aplicadas a qualquer banco; aqui não foi feita conexão.
- Migrações runtime `dbDelta()` e scripts manuais SQL coexistem, sem coordenador global confirmado.
- O rollback varia por migration; algumas não têm par de rollback, e algumas são deliberadamente no-op com instruções para execução manual.
- Compatibilidade de cada DDL com a versão efetiva de MySQL/MariaDB e suporte a CHECK depende de versão; UNVERIFIED.
- Prefixos e substituições de schema para execução precisam ser revisados por operador; valores de configuração não foram lidos.
