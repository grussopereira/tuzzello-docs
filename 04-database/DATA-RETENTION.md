# Retenção, anonimização e exclusão
Propósito: documentar políticas de retenção, anonimização e exclusão evidenciadas no código e nos registros versionados; data: 2026-11-03; este documento descreve o HEAD dos repositórios frontend e backend nessa data.

## Regras encontradas

- Outbox `grpms_wc_status_outbox`: comentário determina retenção append-only e proíbe apagar registros concluídos/mortos; workers podem atualizar metadados de entrega. A migration está marcada LAB ONLY, não aplicada: `backend: sql/auto_complete/002_outbox_and_windows.sql:1-7`.
- Payout holds: comentários dizem conservar holds resolvidos para auditoria e resolver via UPDATE, nunca DELETE (`backend: sql/auto_complete/002_outbox_and_windows.sql:78-79`).
- Rollback de tabelas de histórico/outbox/webhook: exige contagem zero, parada de escritores e recusa de apagar dados para liberar rollback (`backend: sql/auto_complete/001_auto_complete_history.rollback.sql:1-14`; `002_outbox_and_windows.rollback.sql:1-19`; `004_webhook_deliveries.rollback.sql:1-21`). São procedimentos laboratoriais e manuais.
- Auditoria de payout: código prevê remoção compensatória apenas de registros criados pela chamada atual quando falha atualização de pedido, após verificar propriedade/estado (`frontend: includes/financial/grpms-payment.php:1605-1623,1848-1859`). Isso é compensação de fluxo, não política de retenção global.
- KYC guarda URLs de documentos, notas, submissão e revisão (`frontend: includes/admin/kyc/grpms-kyc.php:15-33`). Não foi encontrada política de expurgo ou anonimização nessa definição.
- `grpms_order_locks` é lock temporário com `expires_at`; rotina cria índice por expiração, mas o código/schema consultado não confirma política e frequência de limpeza (`frontend: includes/seller/grpms-seller.php:1192-1205`).

## Política não definida

**TO BE DEFINED:** prazos legais/operacionais por categoria; retenção de pedidos, metadados e registros de reembolso/payout; anonimização de usuários e dados pessoais; exclusão de KYC e arquivos referenciados; expurgo de logs/auditoria/webhooks; retenção de treinamento/certificados; rotina de limpeza de locks expirados; responsável, gatilhos, provas de execução e exceções legais.

**UNVERIFIED:** políticas externas não versionadas, práticas de produção, cron/jobs fora dos arquivos consultados, back-ups e lifecycle dos arquivos remotos. Não foi feita consulta a dados nem a banco.
