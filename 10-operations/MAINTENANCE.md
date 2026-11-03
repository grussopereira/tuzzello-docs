Propósito: documentar rotinas de manutenção verificáveis e identificar as que não têm procedimento comprovado no código.

Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

# Manutenção

## Limite

O código documenta sincronização de pedido via API e o cálculo associado a uma importação (`backend/fastapi_app/routers/checkout/order_router.py:11-20`; `backend/fastapi_app/services/checkout/order_service.py:250-401`). Não foi encontrado procedimento de manutenção operacional aprovado para banco, Stripe, filas, agendamento, retenção ou reprocessamento. Esses itens são **UNVERIFIED**.

## Cuidados durante investigação

- Siga [OPERATIONS-RUNBOOK.md](OPERATIONS-RUNBOOK.md) para a coleta de nomes de tabelas e campos; este documento não autoriza consulta ou alteração em banco.
- Não reexecute `/api/checkout/save-order` manualmente: efeitos de duplicidade, idempotência operacional e segurança de replay para operadores são **UNVERIFIED**.
- Não ative nem reenvie o webhook WooCommerce sem procedimento aprovado. O endpoint está desabilitado quando `service.enabled()` é falso (`backend/fastapi_app/routers/webhooks/woocommerce_order.py:37-44`).
- Não inclua segredos, tokens, conteúdo de `.env`, dados pessoais ou dados de cartão em chamados ou logs.

## Agenda, responsáveis e janelas

Calendário de manutenção, responsáveis, janelas, backup/restore, rollback, critérios de verificação e canais de aprovação: **UNVERIFIED**.
