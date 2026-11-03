Propósito: estabelecer um registro mínimo de triagem de incidentes sem presumir procedimentos não demonstrados pelo código.

Data: 2026-11-03. Este documento descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

# Gestão de incidentes

## Escopo documentado

O caminho observável de sincronização parte do checkout WooCommerce, envia dados à API backend e pode resultar em HTTP 409 ou 500 (`frontend/includes/checkout/grpms-checkout.php:1970-2047`; `backend/fastapi_app/routers/checkout/order_router.py:8-20`). O serviço registra razões de reconciliação (`backend/fastapi_app/services/checkout/order_service.py:23-25`).

## Registro de triagem

Registre horário e fuso, sistema afetado, classe do erro (409, 500, divergência de comissão ou webhook), identificador de pedido mascarado, status HTTP/razão sem segredo, fontes de código consultadas e responsável técnico notificado conforme processo interno existente. Canal e prazo de notificação são **UNVERIFIED**.

Para confirmação Stripe com ausência de pedido ou comissão, siga [OPERATIONS-RUNBOOK.md](OPERATIONS-RUNBOOK.md). A ligação com evento Stripe e o processo de escalonamento são **UNVERIFIED**.

## Contenção e recuperação

Procedimentos de contenção, replay, edição de registros, acionamento de fornecedor, critérios de encerramento e autoridade para aprovar mudanças: **UNVERIFIED**. Não extrapole os comportamentos de endpoint descritos no código para constituir uma operação manual.

## Privacidade

Não registre segredos, dados `.env`, dados de cartão ou dados pessoais. Restrinja evidências a identificadores mascarados e campos técnicos necessários.
