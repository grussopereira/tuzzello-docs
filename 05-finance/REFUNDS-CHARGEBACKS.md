# Reembolsos e chargebacks

Propósito: registrar guardas e operações de reembolso encontradas, separando código integrado de helpers não conectados.

Data: 2026-11-03. Descreve o HEAD de `frontend` (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e `backend` (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Reembolso

O webhook de pedido ignora eventos com status `refunded` ou `partially-refunded` e também payloads com lista `refunds` não vazia; isso não efetua reembolso (`backend/fastapi_app/services/checkout/wc_webhook.py:78-103`). A guarda T2A exige pedido concluído, janela de sete dias após a conclusão, comissão pendente, sem payout, sem refund anterior e sem operação ativa (`backend/fastapi_app/services/checkout/refund_guard.py:49-81`).

`refund_store.py` identifica-se expressamente como helpers SQL “Unwired”; `reject_partial` registra solicitação parcial como `rejected`/`partial_not_allowed` e adiciona auditoria de forma transacional. Declara que não há escritor de reembolso total (`backend/fastapi_app/services/checkout/refund_store.py:1-5,83-90,125-139`).

UNVERIFIED: fluxo efetivo de estorno no provedor, reembolso total, alocação financeira final de taxa/comissão e estorno de comissão já paga.

## Chargebacks

UNVERIFIED: detecção, disputa, prazo, lançamento e distribuição de perdas de chargeback. Nenhuma rotina de chargeback foi identificada nos módulos auditados.
