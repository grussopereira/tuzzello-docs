# Retenção e Descarte de Dados — rascunho

**Propósito:** documentar evidências de persistência e lacunas sobre prazos de retenção e descarte.

**Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nesta data.

> **RASCUNHO:** todo conteúdo legal está sujeito à revisão por profissionais jurídicos qualificados em cada jurisdição.

## Evidência de persistência

O código grava referências de arquivos KYC e metadados em perfil (`frontend/includes/profile/grpms-profile.php:780-801`, `:898-903`), registros de pedido e endereço (`backend/fastapi_app/services/checkout/order_service.py:261-281`, `:419-454`) e taxa de pagamento (`backend/fastapi_app/services/checkout/financial_fee_service.py:37-43`).

Essas gravações demonstram persistência, não um prazo de guarda. Prazo por categoria, gatilho de descarte, exclusão em backups, logs, réplicas, arquivo e integrações estão **UNVERIFIED**. A função `_purge_related_records` contém comandos de exclusão comentados e não comprova rotina ativa de descarte (`backend/fastapi_app/services/checkout/order_service.py:408-416`).

## Decisão pendente

Não há cronograma de retenção verificável nesta revisão. Definir e validar prazos e descarte com responsáveis de dados, operações e assessoria jurídica para cada mercado antes de publicar qualquer compromisso.
