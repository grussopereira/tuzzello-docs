# Auditoria financeira

Propósito: descrever trilhas de auditoria e histórico encontrados nos fluxos financeiros.

Data: 2026-11-03. Descreve o HEAD de `frontend` (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e `backend` (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## O que é registrado

- Webhook: ID da entrega, pedido, tópico, horário de recebimento, resultado, hash do payload e ID da transação; duplicatas são identificadas pela chave primária e verificadas contra os dados originais (`backend/fastapi_app/services/checkout/wc_webhook.py:136-155`). Eventos gravam resultado e logs de commit/rollback (`wc_webhook.py:195-205`).
- Waterfall: loga ativação do flag uma vez e snapshot calculado com ID de pedido e timestamp UTC (`backend/fastapi_app/services/checkout/order_service.py:133-142,235-237`). Reconciliação e bloqueios registram motivo e metadados técnicos (`order_service.py:23-27,538-560`).
- Payout manual: cabeçalho e itens incluem montantes, moeda, operador, ID de transação, status, datas e IDs de administrador/usuário; auditoria permite consulta e exportação CSV (`frontend/includes/financial/grpms-payment.php:1625-1650,2584-2618,2670-2689`).
- Reembolso T2A: a operação rejeitada contém ator/role e a auditoria guarda evento, before/after JSON e horário (`backend/fastapi_app/services/checkout/refund_store.py:125-139`). Esse helper é explicitamente não integrado (`refund_store.py:1-5`).

## O que não é demonstrado

UNVERIFIED: trilha de mudanças para cada edição de valor de comissão, registro de chargeback, comprovante externo de liquidação e auditoria imutável para toda alteração financeira. O log de payout não demonstra por si só confirmação de liquidação pelo provedor.
