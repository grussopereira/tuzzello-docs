Propósito: consolidar bloqueios, conflitos e lacunas que alteram ou impedem o ciclo de venda.

> **Data:** 2026-11-03. Descreve o HEAD dos repositórios frontend (`c1bd75daff802b7ac97bfa32b47f45dd4608d574`) e backend (`e53fb27c2d11e137eafba8096617a20105320026`) nessa data.

## Bloqueios e exceções no código

- **Pedido divergente:** IDs de item, produto, quantidade e totais são conferidos com as linhas WooCommerce; divergências geram erro de integridade/reconciliação. `backend/fastapi_app/services/checkout/order_service.py:42-83`
- **Status incompatível:** atualização aceita apenas transições enumeradas; estado financeiro pago, refund, remoção ou payout bloqueia atualização. `backend/fastapi_app/services/checkout/order_service.py:16-20, 541-587`
- **Comissão inválida:** configuração percentual de plataforma fora do intervalo 0–100 interrompe a importação; link/taxa de afiliado inválidos levam a reconciliação/ausência de atribuição. `backend/fastapi_app/services/checkout/order_service.py:86-127, 145-153`
- **Produto/link não elegível:** produto desativado, estoque esgotado, link não ativo, afiliado sem papel/KYC aprovado ou quantidade incompatível invalidam a validação no checkout. `backend/fastapi_app/services/checkout/checkout_service.py:6-25, 45-89`
- **Reembolso e janela:** regras detalhadas de reembolso são implementadas em módulo específico, mas os limites temporais completos não foram documentados neste conjunto. O gate de conclusão automática filtra razões financeiras e checa a janela via `refund_guard`; **UNVERIFIED** para política comercial completa. `backend/fastapi_app/services/checkout/auto_complete.py:28-48`
- **Envio:** compra de etiqueta EasyPost desabilitada e webhook não configurado no código da rota. `backend/fastapi_app/routers/checkout/easypost_router.py:80-85, 192`
- **Pagamento de comissão:** elegibilidade na interface após 15 dias é visível, mas transferência efetiva não é provada; módulo automático declara não liberar payout. `frontend/includes/financial/grpms-payment.php:197-237`; `backend/fastapi_app/services/checkout/auto_complete.py:1-7`

## UNVERIFIED

1. Transferência efetiva de dinheiro e resposta do provedor de pagamento.
2. Fluxo completo de onboarding, suspensão e reativação de vendedor/afiliado.
3. Enumeração completa de estados de pedido do WooCommerce externo.
4. Política integral de reembolso, chargeback e exceções comerciais.
5. SLA ou processo físico de expedição/entrega.
6. Configuração real de flags ambientais e estado operacional de integrações; não foram lidos `.env` ou segredos.
